# select count (*) from table_name 为什么没有使用主键索引?

由于表中的数据已经有2000多万,在使用select count(*) 查询时，使用explain 查看SQL语句，执行计划中显示使用的**普通索引**，百思不得其解。

在网上查找后遇到一帖

[已解决 select count (*) from table_name 为什么没有使用主键索引?](https://ruby-china.org/topics/26386)

今天在测试索引的时候碰到一个十分诡异的问题，百思不得其解。

表结构
主键索引
token 的索引 devices_token
token 的前置索引 devices_token_prefix_index
我创建了一个只有两个字段、三个索引的测试表。

CREATE TABLE `devices` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `token` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `devices_token` (`token`),
  KEY `devices_token_prefix_index` (`token`(15))
) ENGINE=InnoDB AUTO_INCREMENT=67828774 DEFAULT CHARSET=utf8;
插入了6000万条测试数据后，表的大小为8G。
![](https://l.ruby-china.com/photo/2015/5e3f75a3fd746f9eb204cbf96854d123.jpg)

## 诡异事件
我想看一下数据库已经插入了多少条记录，就做了一个简单的查询

select count(*) from devices
结果整整花了 9s 的时间（这种简单的查询如果使用 primary key 的话，应该只有0.5ms 之内）

我百思不得其解，explain 一下，结果发现这个查询语句压根没有使用 primary key，而是使用了前置索引 devices_token_prefix_index。MySQL 的这种行为怎么这么古怪？
![](https://l.ruby-china.org/photo/2015/074f9092bf92c2a9de154c6e2401fb27.jpg)

我以前彻底的混淆了这两种查询的处理流程，他们两个查询时的策略完全不一样的。

 1.query 1

>--  ms 级别  
>-- token 已经加了索引  
>select count(*) from devices where token="xxx"  
 
 2.query 2

>select count(*) from devices


## Query1
在 Query1中，mysql 会考虑 devices_token, devices_token_prefix_index 两个索引，最终选择了 selectivity 最高的索引 devices_token。

这个查询是 ms 级别的。



## Query2
在 query 2 中，MySQL optimizer 会考虑使用以下三个索引：

primary key 是 clustered index，是所有索引中最大的一个。在这个查询中，无论如何也不会用它。（largest）
* device_token 是次大（ larger）
* device_token_prefix_index 是（large）
* device_token_prefix_index 体积最小，所以会优先用它。

## 结论
在 InnoDB 存储引擎中， primary key 是 clustered index（最大的索引），使用它来处理 select count(*) from table_name 最慢。

>

## MySQL中查找索引信息

在所在的schema中查找中的状态信息
show table status where name ='tablename';
```
[root@mysql.sock][vatti_work_attendance]> show table status where name ='tablename';
+----------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+-----------------------+
| Name           | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time | Check_time | Collation       | Checksum | Create_options | Comment               |
+----------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+-----------------------+
| tablename | InnoDB |      10 | Dynamic    |    0 |              0 |       16384 |               0 |            0 |         0 |           NULL | 2019-02-28 17:03:58 | NULL        | NULL       | utf8_general_ci |     NULL |                | NULL        |
+----------------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+-----------------------+
1 row in set (0.00 sec)

```
查看MySQL表中索引信息

## MySQL索引统计信息更新相关的参数
 

MySQL统计信息相关的参数：

 

　　1. innodb_stats_on_metadata（是否自动更新统计信息），MySQL 5.7中默认为关闭状态

　　　　仅在统计信息配置为非持久化的时候生效。
　　　　也就是说在innodb_stats_persistent 配置为OFF的时候，非持久化存储统计信息的手，innodb_stats_on_metadata的设置才生效。
　　　　当innodb_stats_on_metadata设置为ON的时候，
　　　　InnoDB在执show table status 或者访问INFORMATION_SCHEMA.TABLES
　　　　或者INFORMATION_SCHEMA.STATISTICS 系统表的时候，
　　　　更新费持久化统计信息（类似于ANALYZE TABLE），innodb_stats_on_metadata不管打开还是关闭，都不影响持久化存储统计信息的索引
　　　　某个索引的统计信息更新时间参考mysql.innodb_index_stats这个系统表
       

　　2. innodb_stats_auto_recalc

　　　　是否自动触发更新统计信息，仅影响持久化存储的统计信息的表，阈值是变化的数据超过表行数的10%。
　　　　也就是说，一个表索引统计信息是持久化存储的，并且表中数据变化了超过10%，
　　　　如果innodb_stats_auto_recalc为ON，就会自动更新统计信息，否则不会自动更新

　　3. innodb_stats_persistent（非持久化统计信息开关），MySQL 5.7中默认为打开，持久化存储统计信息

　　　　该选项设置为ON时候，统计信息会持久化存储到磁盘中，而不是存在在内存中，
　　　　相反，如果是非持久化存储的（存在内存中），相应的统计信息会随着服务器的关闭而丢失。

　　4. innodb_stats_persistent_sample_pages （持久化更新统计信息时候索引页的取样页数）

　　　　默认是20个page，如果设置的过高，那么在更新统计信息的时候，会增加ANALYZE TABLE的执行时间。

　　5. innodb_stats_transient_sample_pages（临时性更新统计信息时候索引页的取样页数）

　　　　默认值是8，innodb_stats_persistent设置为disable的情况下，也即非持久化明确关闭的时候，innodb_stats_transient_sample_pages才生效，
　　　　也就是非持久化存储过索引统计信息的时候，innodb_stats_transient_sample_pages为更新统计信息的采样页数
　　　　这个值是否生效，要依赖于innodb_stats_on_metadata，而innodb_stats_on_metadata又依赖于innodb_stats_persistent
　　　　总而言之：如果配置为持久化存储统计信息，非持久化相关的配置选项就不起作用

　　6. innodb_stats_sample_pages

　　　　已弃用. 已用innodb_stats_transient_sample_pages 替代。
　　　　为啥要用innodb_stats_transient_sample_pages替代？
　　　　个人猜测是一开始参数命名不规范，既然是临时行统计信息，却没有做到见名知意，与innodb_stats_persistent_sample_pages区分开来，
　　　　或许是一开始MySQL中只有临时行统计信息，没有持久化统计信息。

 

统计信息更新测试1：打开innodb_stats_auto_recalc的情况下，统计信息会在触发其更新阈值后自动更新


>select * from mysql.innodb_index_stats where table_name = 'tablename';

在看看到CSDN的一个博客帖子后,心中豁然开朗,虽然是**SQL Server**但是对聚集索引的理解更深入一些
![通过非聚集索引让select count(*) from 的查询速度提高几十倍、甚至千倍](https://blog.csdn.net/sqlserverdiscovery/article/details/12646371)
以下内容转自csdn*不想长大啊*

通过非聚集索引，可以显著提升count(*)查询的性能。

有的人可能会说，这个count(*)能用上索引吗，这个count(*)应该是通过表扫描来一个一个的统计，索引有用吗？



不错，一般的查询，如果用索引查找，也就是用Index Seek了，查询就会很快。

 

之所以快，是由于查询所需要访问的数据只占整个表的很小一部分，如果访问的数据多了，那反而不如通过表扫描来的更快，因为扫描用的是顺序IO，效率更高，比运用随机IO访问大量数据的效率高很多。

 

相应的，如果只需要访问少量数据，那么索引查找的效率远高于表扫描，因为通过随机IO来访问少量数据的效率远高于通过顺序IO来访问少量数据，之所以扫描的效率较低是由于扫描访问了很多不需要的数据。

 

那么，通过非聚集索引，提升select count(*) from 的查询速度的本质在于，非聚集索引所占空间的大小往往，远小于聚集索引或堆表所占用的空间大小；

同样的，表中占用较少字节的字段的非聚集索引，对于速度的提升效果，也要远大于，占用较多字节的字段的非聚集索引，因为占用字节少，那么索引占用的空间也少，同样是扫描，只需要更少的时间，对硬盘的访问次数也更少，那么速度就会更快了。



下面通过一个实验，来说明非聚集索引为什么能提高count（*）的查询速度。



1、建表，插入数据

if OBJECT_ID('test') is not null
   drop table test
go
 
create table test
(
id int identity(1,1),
vid int ,
v varchar(600),
constraint pk_test_id primary key (id)
)
go
 
 
 
insert into test(vid,v)
select 1,REPLICATE('a',600) union all
select 2,REPLICATE('b',600) union all
select 3,REPLICATE('c',600) union all
select 4,REPLICATE('d',600) union all
select 5,REPLICATE('e',600) union all
select 6,REPLICATE('f',600) union all
select 7,REPLICATE('g',600) union all
select 8,REPLICATE('h',600) union all
select 9,REPLICATE('i',600) union all
select 10,REPLICATE('j',600)
go
 
 
--select POWER(2,18) * 10
--2621440条数据
begin tran
	insert into test(vid,v)
	select vid,v
	from test
commit
go 18
 
 
--建立非聚集索引
create index idx_test_vid on test(vid)
 


2、查看采用聚集索引和非聚集索引后，查询的资源消耗

--输出详细的IO和时间(cpu、流逝的时间)上的开销信息
set statistics io on
set statistics time on
 
 
/* 采用聚集索引
SQL Server 分析和编译时间: 
   CPU 时间 = 0 毫秒，占用时间 = 0 毫秒。
(1 行受影响)
表 'test'。扫描计数 5，逻辑读取 206147 次，物理读取 0 次，预读 0 次，lob 逻辑读取 0 次，lob 物理读取 0 次，lob 预读 0 次。
 SQL Server 执行时间:
   CPU 时间 = 921 毫秒，占用时间 = 277 毫秒。
*/
select COUNT(*)
from test with(index (pk_test_id))
 
 
 
/*采用非聚集索引
SQL Server 分析和编译时间: 
   CPU 时间 = 0 毫秒，占用时间 = 1 毫秒。
(1 行受影响)
表 'test'。扫描计数 5，逻辑读取 4608 次，物理读取 0 次，预读 0 次，lob 逻辑读取 0 次，lob 物理读取 0 次，lob 预读 0 次。
 SQL Server 执行时间:
   CPU 时间 = 327 毫秒，占用时间 = 137 毫秒。
*/
select count(*)
from test with(index (idx_test_vid))


3、从上面的开销可以看出：

a、通过聚集索引来查询count(*)时，逻辑读取次数206147次，执行时间和占用时间分别是921毫秒和277毫秒，从执行计划中看出，其查询开销是96%。

b、非聚集索引的逻辑读取次数是4608次，而执行时间和占用时间是327毫秒和137毫秒，查询开销是4%。

c、表扫描的逻辑读取次数是201650次，执行时间和占用时间是765毫秒和233毫秒。



这里需要注意的是，由于两个执行计划都采用了并行计划，导致了执行时间远大于占用时间，这主要是因为执行时间算的是多个cpu时间的总和，我的笔记本电脑有4个cpu，那么921/4 大概就是230毫秒左右，也就是每个cpu花在执行上的时间大概是230毫秒左右，和277毫秒就差不多了。



从这些开销信息可以看出，非聚集索引的逻辑读取次数是聚集索引的50分之一，执行时间是聚集索引的2-3分之一左右，查询开销上是聚集索引的24分之一。



很有意思的是，表扫描的逻辑读取次数要比聚集索引的要少4497次，这个逻辑读取次数201650，是可以查到，看下面的代码：
之所以能查到，是因为全表扫描，无非就是把表中所有的页，都扫描一遍，所以扫描的次数正好是表中的页数201650.

4、那为什么非聚集索引来查询count(*) 的效率是最高的呢？

其实上面分别提到了，通过聚集索引、非聚集索引、表扫描，3种方式来查询，从执行计划可以看出来，3种方式都是扫描，那为什么非聚集索引效率最高？

其实，很简单，谁扫描的次数少，也就是扫描的页数少，那谁的效率当然就高了。



看下面的代码，就明白了：


use master
go
 
--index_id为1表示聚集索引
select index_id,
       index_type_desc,
       alloc_unit_type_desc,
       page_count                --201650
from sys.dm_db_index_physical_stats
(
db_id('wcc'),object_id('wcc.dbo.test'),1,null,'detailed'
)d
where index_level = 0  --只取level为0的,也就是页子级别
 
/*
index_id	index_type_desc		alloc_unit_type_desc   page_count
1	        CLUSTERED INDEX	    IN_ROW_DATA	           201650
*/
 
 
 
--index_id为2的,表示非聚集索引
select index_id,
       index_type_desc,
       alloc_unit_type_desc,
       page_count               --4538
from sys.dm_db_index_physical_stats
(
db_id('wcc'),object_id('wcc.dbo.test'),2,null,'detailed'
)d
where index_level = 0
 
/*
index_id	index_type_desc		alloc_unit_type_desc	page_count
2			NONCLUSTERED INDEX	IN_ROW_DATA				4538
*/

聚集索引的叶子节点的页数是201650，而非聚集索引的 叶子节点的页数是4538，差了近50倍，而在没有索引的时候，采用表扫描时，叶子节点的页数是201650，与聚集索引一样。


效率的差异不仅在与逻辑读取次数，因为逻辑读取效率本身是很高的，是直接在内存中读取的，但SQL Server的代码需要扫描内存中的数据201650次，也就是循环201650次，可想而知，cpu的使用率会暴涨，会严重影响SQL Server处理正常的请求。



假设这些要读取的页面不在内存中，那问题就大了，需要把硬盘上的数据读到内存，关键是要读201650页，而通过索引只需要读取4538次，效率的差距就会更大。



另外，实验中只是200多万条数据，如果实际生产环境中有2亿条记录呢？到时候，效率的差距会从几十倍上升到几百倍、几千倍。

 

5、那是不是只要是非聚集索引，都能提高select count(*) from查询的效率吗？

这个问题是由下面的网友提出的问题，而想到的一个问题。

如果按照v列来建索引，而v列的数据类型是varchar(600)，所以这个新建的索引，占用的页数肯定是非常多的，应该仅次于聚集索引的201650页，那么完成索引扫描的开销肯定大于，按vid列建立的非聚集索引，而vid的数据类型是int。

所以，不是只要是非聚集索引，就能提高查询效率的。

 

总结一下：

执行select count(*) from查询的时候，要进行扫描，有人可能会说，扫描性能很差呀，还能提高性能？那么，难道用索引查找吗？这样性能只会更差。

这里想说的是，没有最好的技术，只有最适合的技术，要想提高这种select count(*) from查询的性能，那就只能用扫描。

这里，要提高效率的关键，就是减少要扫描的页数，而按照占用字节数少的字段，来建立非聚集索引，那么这个非聚集索引所占用的页数，远远少于聚集索引、按占用字节数较多的列建立的非聚集索引，所占用的页数，这样就能提高性能了。



最后，有两个关于索引的帖子，不错：


两个问题：1，（聚集或者非聚集的）索引页会不会出现也拆分；2，非聚集索引存储时又没排序：

http://bbs.csdn.net/topics/390594730



继续：非聚集索引行在存储索引键时确实是排序了的，用事实说话，理论+实践：

http://bbs.csdn.net/topics/390595949

