# Where MySQL Stores Temporary Files
MySQL 临时文件的作用
临时文件可以多路径使用不通的磁盘设备,目的是避免单个磁盘IO瓶颈导致临时表文件排序SQL执行慢 

 Linux以冒号（:）分割   
 以下是MySQL配置文件tmpdir 参数，此 参数是一个静态文件参数需要重启MySQL服务器后生效  
 ```
 > tmpdir=/tmp:/data/tmp
 
 [root@mysql.sock][(none)]> show variables like '%tmpdir%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| innodb_tmpdir     |       |
| slave_load_tmpdir | /tmp  |
| tmpdir            | /tmp  |
+-------------------+-------+
3 rows in set (0.00 sec)

[root@mysql.sock][(none)]> 

1、vim /etc/my.cnf
tmpdir=/tmp:/data/tmp

2、/etc/init.d/mysqld restart

[root@mysql.sock][(none)]> show variables like '%tmpdir%';
+-------------------+----------------+
| Variable_name     | Value          |
+-------------------+----------------+
| innodb_tmpdir     |                |
| slave_load_tmpdir | /tmp           |
| tmpdir            | /tmp:/data/tmp |
+-------------------+----------------+
3 rows in set (0.00 sec)

[root@mysql.sock][(none)]> 


3、lsof +L1 查看文件
[root@jst-centos-testdb-130 tmp]#  lsof +L1
COMMAND     PID      USER   FD   TYPE DEVICE SIZE/OFF NLINK    NODE NAME
hald       1627 haldaemon  txt    REG  253,0   359184     0 1052599 /usr/sbin/hald (deleted)
hald-runn  1628      root  txt    REG  253,0    23304     0 1054852 /usr/libexec/hald-runner (deleted)
hald-addo  1660      root  txt    REG  253,0    31448     0 1054679 /usr/libexec/hald-addon-input.#prelink#.PnLpYG (deleted)
hald-addo  1671 haldaemon  txt    REG  253,0    21096     0 1055574 /usr/libexec/hald-addon-acpi.#prelink#.yK66ns (deleted)
mysqld    33398     mysql   29u   REG  253,0        0     0 2883595 /data/tmp/ibDbAgEN (deleted)
mysqld    33398     mysql   30u   REG  253,0        0     0 6291458 /tmp/ibAs5vXn (deleted)
mysqld    33398     mysql   31u   REG  253,0        0     0 2883596 /data/tmp/ib8vwLgY (deleted)
mysqld    33398     mysql   32u   REG  253,0        0     0 2883600 /data/tmp/ibbkbJKd (deleted)
mysqld    33398     mysql   36u   REG  253,0        0     0 6291459 /tmp/ibALI53S (deleted)
[root@jst-centos-testdb-130 tmp]#  lsof +L1

 window以分号（;）分割
```

MySQL 5.7官方手册  
[Where MySQL Stores Temporary Files](https://dev.mysql.com/doc/refman/5.7/en/temporary-files.html)  
[MySQL中的内部临时表使用](https://dev.mysql.com/doc/refman/5.7/en/internal-temporary-tables.html)
