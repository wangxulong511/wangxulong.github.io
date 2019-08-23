# ProxySQL+MGR单主
> https://blog.csdn.net/L835311324/article/details/91126335
## ProxySQL github
> https://github.com/sysown/proxysql
##  ProxySQL github 中文手册(马龙帅翻译)
> https://github.com/malongshuai/proxysql/wiki

## ProxySQL实现MySQL读写分离
> https://www.jianshu.com/p/121e3196cf29

## Mysql MGR监控及优化点
> http://blog.itpub.net/9522838/viewspace-2285017/



ProxySQL  管理端口默认（6032）
MySQL   连接端口（6033）
```
 admin_variables=
{
	admin_credentials="admin:admin"
#	mysql_ifaces="127.0.0.1:6032;/tmp/proxysql_admin.sock"
	mysql_ifaces="0.0.0.0:6032"
#	refresh_interval=2000
#	debug=true
}

mysql_variables=
{
	threads=4
	max_connections=2048
	default_query_delay=0
	default_query_timeout=36000000
	have_compress=true
	poll_timeout=2000
#	interfaces="0.0.0.0:6033;/tmp/proxysql.sock"
	interfaces="0.0.0.0:6033"
	default_schema="information_schema"
	stacksize=1048576
	server_version="5.5.30"
	connect_timeout_server=3000
  ...
}
```

第一次搭建时一直使用6032端口连接，一直报如下错误,发生这样的问题主要还是没有对proxysql的架构进行了解，没有详细的查看
```
[root@mgr-1 ~]# mysql -uproxysql -pproxysql -h 127.0.0.1 -P 6032 -e 'select @@server_id '
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1045 (28000): ProxySQL Error: Access denied for user 'proxysql'@'127.0.0.1' (using password: YES)
[root@mgr-1 ~]# 
[root@mgr-1 ~]# mysql -uproxysql -pproxysql -h 127.0.0.1 -P 6033 -e 'select @@server_id '
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------+
| @@server_id |
+-------------+
|     1923306 |
+-------------+
[root@mgr-1 ~]# 
[root@mgr-1 ~]# mysql -uproxysql -pproxysql -h 127.0.0.1 -P 6033 -e 'select @@server_id '
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------+
| @@server_id |
+-------------+
|     1933306 |
+-------------+
```

## mysql 5.7 group replication 之三 ERROR 3092 (HY000): The server is not configured properly to be an ac
> https://blog.csdn.net/ctypyb2002/article/details/88062847

关闭MySQL数据库后重启mgr发现启动不了 ,从节点执行后可以启动
set global group_replication_allow_local_disjoint_gtids_join=ON; 
```
[root@mysql.sock][(none)]> START GROUP_REPLICATION;
ERROR 3092 (HY000): The server is not configured properly to be an active member of the group. Please see more details on error log.
```

