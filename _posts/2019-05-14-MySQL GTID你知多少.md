# MySQL GTID你知多少
> https://www.cnblogs.com/gomysql/p/7417995.html


MySQL在5.6的版本推出了GTID复制，相比传统的复制，GTID复制对于运维更加友好，这个事务是谁产生多少事务，执行后直接的标识出来，当然GTID也有限制，对于什么是GTID可以参考我之前的文章：MySQL 5.6 GTID Replication，那么今天主要是想和同学们分享一下关于从库show slave status中的Retrieved_Gtid_Set和Executed_Gtid_Set。

Retrieved_Gtid_Set：从库已经接收到主库的事务编号  
Executed_Gtid_Set：已经执行的事务编号  

以下的gitd 是因为级联复制后从库执行的GTID值  
Retrieved_Gtid_Set: 8448095b-2779-11e8-9eee-801844f2eb44:24782795-43457076
Executed_Gtid_Set: 8448095b-2779-11e8-9eee-801844f2eb44:1-43457076,
b29adcee-d60a-11e6-8697-0050568c3815:1-151573105,
ec3d8ee4-da32-11e6-a1b2-0050568c769d:1-6,
f653199e-f75b-11e8-aa82-20040fea8c80:1-2

