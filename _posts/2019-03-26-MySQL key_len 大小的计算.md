#  MySQL key_len 大小的计算

[MySQL key_len 大小的计算](https://www.cnblogs.com/zhoujinyi/p/3784450.html)  

背景：

   当用Explain查看SQL的执行计划时，里面有列显示了 key_len 的值，根据这个值可以判断索引的长度，在组合索引里面可以更清楚的了解到了哪部分字段使用到了索引。

环境：
```

CREATE TABLE `tmp_0612` (
  `id` int(11) NOT NULL,
  `name` varchar(10) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

插入数据：

insert into tmp_0612 values(1,'a',11,'hz'),(2,'b',22,'gz'),(3,'c',33,'aa');
创建索引：

alter table tmp_0612 add index idx_name(name);
alter table tmp_0612 add index idx_age(age);
测试：

explain select * from tmp_0612 where name ='a';
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key      | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_name      | idx_name | 33      | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
从上面的执行计划可知，索引的长度是33。比预想的30（10*3(utf8)）要高出3字节，为什么呢？进一步测试：

修改name 成 not null

alter table tmp_0612 modify name varchar(10) not null;
再看执行计划：

explain select * from tmp_0612 where name ='a';
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key      | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_name      | idx_name | 32      | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
发现上面的执行计划和第一次的有区别(key_len)，经过多次测试，发现字段允许NULL的会多出一个字节。想到了之前的一篇文章，NULL是需要一个标志位的，占用1个字符。那还有2个字节怎么算？这里想到的是会不会和 多字节字符集 相关？那改字符集试试：


alter table tmp_0612 convert to charset latin1;

explain select * from tmp_0612 where name ='a';
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key      | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_name      | idx_name | 12      | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+

发现还是多了2个字节，看来和多字节字符集没有关系了。那会不会和 变长字段 有关系？再试试：


alter table tmp_0612 convert to charset utf8;

alter table tmp_0612 modify name char(10) not null;

explain select * from tmp_0612 where name ='a';
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key      | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_name      | idx_name | 30      | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+

和预料的一样了，是30=10*3。到这里相信大家都已经很清楚了，要是还比较模糊就看反推到33字节。

改成允许NULL，应该会变成31。


alter table tmp_0612 modify name char(10);

explain select * from tmp_0612 where name ='a';
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key      | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_name      | idx_name | 31      | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+

改成变长字段类型，应该会变成33。


alter table tmp_0612 modify name varchar(10);

explain select * from tmp_0612 where name ='a';
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key      | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_name      | idx_name | 33      | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+

改成单字节字符集，还是还是需要额外的3字节（1：null ；变长字段：2），和字符集无关。


alter table tmp_0612 convert to charset latin1;

explain select * from tmp_0612 where name ='a';
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
| id | select_type | table    | type | possible_keys | key      | key_len | ref   | rows | Extra                 |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_name      | idx_name | 13      | const |    1 | Using index condition |
+----+-------------+----------+------+---------------+----------+---------+-------+------+-----------------------+

反推上去都和预测的一样。

其他测试：


explain select * from tmp_0612 where age = 11;
+----+-------------+----------+------+---------------+---------+---------+-------+------+-------+
| id | select_type | table    | type | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+----------+------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_age       | idx_age | 5       | const |    1 | NULL  |
+----+-------------+----------+------+---------------+---------+---------+-------+------+-------+

alter table tmp_0612 modify age int not null;

explain select * from tmp_0612 where age = 11;
+----+-------------+----------+------+---------------+---------+---------+-------+------+-------+
| id | select_type | table    | type | possible_keys | key     | key_len | ref   | rows | Extra |
+----+-------------+----------+------+---------------+---------+---------+-------+------+-------+
|  1 | SIMPLE      | tmp_0612 | ref  | idx_age       | idx_age | 4       | const |    1 | NULL  |
+----+-------------+----------+------+---------------+---------+---------+-------+------+-------+

int 是占4个字符的，上面key_len的也是符合预期。关于组合索引的，可以自己去测试玩。
```
总结：

   变长字段需要额外的2个字节，固定长度字段不需要额外的字节。而null都需要1个字节的额外空间，所以以前有个说法：索引字段最好不要为NULL，因为NULL让统计更加复杂，并且需要额外一个字节的存储空间。


key_len的长度计算公式：

varchr(10)变长字段且允许NULL      : 10*(Character Set：utf8=3,gbk=2,latin1=1)+1(NULL)+2(变长字段)
varchr(10)变长字段且不允许NULL    : 10*(Character Set：utf8=3,gbk=2,latin1=1)+2(变长字段)

char(10)固定字段且允许NULL        : 10*(Character Set：utf8=3,gbk=2,latin1=1)+1(NULL)
char(10)固定字段且不允许NULL        : 10*(Character Set：utf8=3,gbk=2,latin1=1)
