## ANALYZE TABLE 
用来分析和存储表的关键字的分布，使得系统获得准确的统计信息，影响 SQL 的执行计划的生成。对于数据基本没有发生变化的表，是不需要经常进行表分析的。但是如果表的数据量变化很明显，用户感觉实际的执行计划和预期的执行计划不同的时候，执行一次表分析可能有助于产生预期的执行计划。
表分析的语法是：
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
这个命令对 MyISAM，BDB，InnoDB 存储引擎的表有作用。对于 MyISAM 存储引擎的表，这个命令的执行效果与 myisamchk –a 的执行效果相当。

## CHECK TABLE 
的作用是检查表或者视图是否存在错误，对 MyISAM 和 InnoDB 存储引擎的表有作用。对于 MyISAM 存储引擎的表进行表检查，也会同时更新关键字统计数据。
表检查的语法是：
CHECK TABLE tbl_name [, tbl_name] ... [option] ... option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}

## CHECKSUM TABLE 
数据在传输时可能会发生变化，也有可能因为其他原因损坏，为了保证数据的一致，我们可以计算校验值
用法：CHECKSUM TABLE tbl_name [, tbl_name] ...[QUICK|EXTENDED ]
QUICK表示返回存储的checksum值，而checksum会重新计算checksum值，如果没有指定选项，默认是extended

## OPTIMIZE TABLE
语法：OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
如果你的MySQL是有备库的，如果你只希望在主库上执行的话，那么可以加上关键字NO_WRITE_TO_BINLOG（或者LOCAL）。
作用：删除表的一大部分数据后，或者对含有可变长度行的表（含有VARCHAR, BLOB或TEXT列的表）进行了很多更改，使用Optimize后能帮我们回收更多的磁盘空间、减少“碎片” （defragment）。
原理：当你删除数据时，mysql并不会回收，被已删除数据的占据的存储空间，以及索引位。而是空在那里，而是等待新的数据来弥补这个空缺，这样就有一个缺点，如果一时半会，没有数据来填补这个空缺，那这样就太浪费资源了。所以对于写比较频烦的表，要定期进行optimize，帮我们回收更多的空间、减少“碎片” （defragment）。一个月一次，看实际情况而定了。
适用的的存储引擎：MyISAM, InnoDB, BDB
对于BDB表，OPTIMIZE TABLE目前被映射到ANALYZE TABLE上。
对于InnoDB表，OPTIMIZE TABLE被映射到ALTER TABLE上，这会重建表。重建操作能更新索引统计数据并释放成簇索引中的未使用的空间，回收主键索引空间。
适用场景：磁盘耗尽、或者InnoDB Tablespaces用完的情况。这时候，在考虑扩容等方案之前，最好先使用Optimize Table试试。
注意，在OPTIMIZE TABLE运行过程中，MySQL会锁定表。
```
OPTIMIZE TABLE 执行期间在执行 ANALYZE TABLE 会导致select 语句长时间等待 （Waiting for table flush）
```


# 实践总结

    -- 不需要经常运行，删除大量数据、更新大量变长数据时也仅仅需要每周后者一月运行一次
    -- 在执行OPTIMIZE TABLE 时不要执行 ANALYZE TABLE 
