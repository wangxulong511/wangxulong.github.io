# Oracle中shrink space命令详解
> https://www.cnblogs.com/einyboy/archive/2012/08/10/2631464.html


## oracle回收表空间
```
SELECT Upper(F.TABLESPACE_NAME)         "表空间名",
       D.TOT_GROOTTE_MB                 "表空间大小(M)",
       D.TOT_GROOTTE_MB - F.TOTAL_BYTES "已使用空间(M)",
       To_char(Round(( D.TOT_GROOTTE_MB - F.TOTAL_BYTES ) / D.TOT_GROOTTE_MB * 100, 2), '990.99')
       || '%'                           "使用比",
       F.TOTAL_BYTES                    "空闲空间(M)",
       F.MAX_BYTES                      "最大块(M)"
FROM   (SELECT TABLESPACE_NAME,
               Round(Sum(BYTES) / ( 1024 * 1024 ), 2) TOTAL_BYTES,
               Round(Max(BYTES) / ( 1024 * 1024 ), 2) MAX_BYTES
        FROM   SYS.DBA_FREE_SPACE
        GROUP  BY TABLESPACE_NAME) F,
       (SELECT DD.TABLESPACE_NAME,
               Round(Sum(DD.BYTES) / ( 1024 * 1024 ), 2) TOT_GROOTTE_MB
        FROM   SYS.DBA_DATA_FILES DD
        GROUP  BY DD.TABLESPACE_NAME) D
WHERE  D.TABLESPACE_NAME = F.TABLESPACE_NAME
ORDER  BY 1

在Oracle中，经常有这样的情况，由于误操作，使某个表空间过大。delete 方法不会清除高水位线，用了truncate之后，虽然高水位线已经清除，但是扩充的表空间并没有缩小，所以应该用下面的方法进行缩小：
1.查找表空间对应的数据文件id，以USERS表空间为例：
select FILE_ID from dba_data_files where tablespace_name ='USERS'
2.允许表空间进行收缩（对表空间进行融合）：
alter tablespace users coalesce;
3.查询表空间中表对应的大小，好确定对哪张表进行操作：
select * from dba_segments where tablespace_name='USERS' and segment_type='TABLE'
4.允许表进行行移动（某些表不能进行truncate，只能delete）
alter table test enable row movement;
5.对表进行高水位线回收：
alter table test shrink space;
6.最后，对第一步查出的数据文件id进行压缩（resize），大小一定要大于已用大小：
alter database datafile 4 resize 2000M;


--07.回收表空间碎片
analyze table test compute statistics;

```
## ORACLE表空间和表碎片分析及整理方法
> https://blog.csdn.net/x6_9x/article/details/50596589

## oracle管理笔记
> http://blog.itpub.net/14578680/viewspace-608318/
