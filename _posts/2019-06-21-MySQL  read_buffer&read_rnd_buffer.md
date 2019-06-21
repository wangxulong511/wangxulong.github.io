 SHOW VARIABLES
WHERE
 variable_name IN (
  'read_buffer_size',
  'read_rnd_buffer_size',
  'sort_buffer_size',
  'join_buffer_size',
  'binlog_cache_size',
  'tmp_table_size'
 );
 
 
 read_buffer&read_rnd_buffer
分别存放了对顺序和随机扫描（例如按照排序的顺序访问）的缓存。当thread进行顺序或随机扫描数据时会首先扫描该buffer空间以避免更多的物理读。每个sessionRDS给予256K的大小。
 sort_buffer
需要执行order by和group by的sql都会分配sort_buffer用来存储排序的中间结果，当排序的过程中如果存储带下大于sort_buffer_size的话会在磁盘生成临时表以完成操作。根据MySQL的文档可知在linux系统中，当分配空间大于2M时会使用mmap() 而不是 malloc() 来进行内存分配，导致效率降低。在RDS上给予256K。
 join_buffer
MySQL仅支持nest loop的join算法，处理逻辑是驱动表的一行和非驱动表联合查找，这时就可以将非驱动表放入join_buffer，不需要访问拥有并发保护机制的buffer_pool。RDS给予256K大小。
 binlog_cache
该区域用来缓存该thread的binlog日志。在一个事务还没有commit之前会先将其日志存储于binlog_cache中，等到事务commit后会将其binlog刷回磁盘上的binlog文件以持久化。同样该大小为256K。
 tmp_table
不同于上面的各个session层次的buffer，这个参数是可以在控制台上修改。是指用户内存临时表的大小，如果该thread创建的临时表超过它设置的大小会把临时表转换为磁盘上的一张MyISAM临时表。如果用户在执行事务的时候遇到类似“”这样的错误的时候可以考虑将其修改更大一些。
[Err] 1114 - The table '/home/mysql/data3081/tmp/#sql_6197_2' is full
