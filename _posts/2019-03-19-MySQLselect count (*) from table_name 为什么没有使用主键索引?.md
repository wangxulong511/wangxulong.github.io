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
```
MySQL索引统计信息更新相关的参数
 

MySQL统计信息相关的参数：

 

　　1. innodb_stats_on_metadata（是否自动更新统计信息），MySQL 5.7中默认为关闭状态

　　　　仅在统计信息配置为非持久化的时候生效。
　　　　也就是说在innodb_stats_persistent 配置为OFF的时候，非持久化存储统计信息的手，innodb_stats_on_metadata的设置才生效。
　　　　当innodb_stats_on_metadata设置为ON的时候，
　　　　InnoDB在执show table status 或者访问INFORMATION_SCHEMA.TABLES
　　　　或者INFORMATION_SCHEMA.STATISTICS 系统表的时候，
　　　　更新费持久化统计信息（类似于ANALYZE TABLE），innodb_stats_on_metadata不管打开还是关闭，都不影响持久化存储统计信息的索引
　　　　某个索引的统计信息更新时间参考mysql.innodb_index_stats这个系统表

某个索引的统计信息更新时间参考mysql.innodb_index_stats这个系统表
select * 
from mysql.innodb_index_stats 
where table_name = 'teststatistics';

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

 
```

>select * from mysql.innodb_index_stats where table_name = 'tablename';
