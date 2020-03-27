# mysql 用 group by 查询时，会自动保留   对应组 ‘最先搜索出来的数据’，但这时数据可能不是最新的
> http://www.iamlintao.com/6955.html

如何才能获取‘最新数据’ 呢，这需要根据mysql的版本来“区别对待”

原因：在mysql5.7中，如果不加limit，系统会把order by优化掉。

在mysql5.7手册的8.2.2.1中有解释：

子查询的优化是使用半连接的策略完成的(The optimizer uses semi-join strategies to improve subquery execution)

使用半连接进行优化，子查询语句必须满足一些标准(In MySQL, a subquery must satisfy these criteria to be handled as a semi-join)。

其中一个标准是:必须不是一个包含了limit和order by的语句(It must not have ORDER BY with LIMIT.)

对于mysql 5.5版本

select * from (

select * from table_name order by create_time desc

) as t
group by t.id;
EOT;

对于mysql 5.7版本,需要加入limit限制，否则不生效

select * from (

select * from table_name order by create_time desc limit 100000

) as t
group by t.id;
EOT;
