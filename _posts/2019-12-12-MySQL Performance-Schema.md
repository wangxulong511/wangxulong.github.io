# 数据库对象事件与属性统计 | performance_schema全方位介绍（五）
> https://blog.csdn.net/n88Lpo/article/details/82700908
```
详细介绍了performance_schema的事件统计表，但这些统计数据粒度太粗，仅仅按照事件的5大类别+用户、线程等维度进行分类统计，
但有时候我们需要从更细粒度的维度进行分类统计，例如：某个表的IO开销多少、锁开销多少、以及用户连接的一些属性统计信息等。
此时就需要查看数据库对象事件统计表与属性统计表了。今天将带领大家一起踏上系列第五篇的征程(全系共7个篇章)，本期将为大家
全面讲解performance_schema中对象事件统计表与属性统计表。下面，请跟随我们一起开始performance_schema系统的学习之旅吧~
```
## MySQL Performance-Schema(三) 实践篇
> https://www.cnblogs.com/cchust/p/5061131.html  
> https://www.cnblogs.com/cchust/category/802863.html
