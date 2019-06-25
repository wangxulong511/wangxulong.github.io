# Last_Error: Relay log read failure: Could not parse relay log event entry.

服务器磁盘不足导致从库复制中断


[test@centos3 ~]$ df -lh
Filesystem                      Size  Used Avail Use% Mounted on
/dev/mapper/vg_centos-LogVol01  592G  562G   13M 100% /
tmpfs                           3.9G     0  3.9G   0% /dev/shm
/dev/sda1                       485M   39M  421M   9% /boot


## 1、版本  
###1）服务器版本  
CentOS release 6.5 (Final)

[test@centos3 ~]$ uname -a  
Linux jst-101-centos3 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
###2）数据库版本
```
mysql> status
--------------
mysql  Ver 14.14 Distrib 5.6.32, for linux-glibc2.5 (x86_64) using  EditLine wrapper

Connection id:		17
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		5.6.32-log MySQL Community Server (GPL)
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/tmp/mysql.sock
Uptime:			4 days 21 hours 52 min 5 sec

Threads: 3  Questions: 1412123  Slow queries: 0  Opens: 151  Flush tables: 1  Open tables: 140  Queries per second avg: 3.327
--------------

mysql> 

```
## 2.问题描述

### 2.1 发现问题

虚拟机磁盘不足，导致数据库复制异常。查看mysql从库状态，show slave status\G;发现如下报错：

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.101.222.138
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: bin.000928
          Read_Master_Log_Pos: 172849045
               Relay_Log_File: jst-101-centos3-relay-bin.001815
                Relay_Log_Pos: 293704
        Relay_Master_Log_File: bin.000906
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1594
                   Last_Error: Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave.
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 293586
              Relay_Log_Space: 30975039218
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 356436
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 1594
               Last_SQL_Error: Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave.
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1783306
                  Master_UUID: 8448095b-2779-11e8-9eee-801844f2eb44
             Master_Info_File: /data/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Reading event from the relay log
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 190624 14:06:14
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 8448095b-2779-11e8-9eee-801844f2eb44:25153820-48854161
            Executed_Gtid_Set: 8448095b-2779-11e8-9eee-801844f2eb44:1-48404438,
b29adcee-d60a-11e6-8697-0050568c3815:1-151573105,
ec3d8ee4-da32-11e6-a1b2-0050568c769d:1-6
                Auto_Position: 1
1 row in set (0.00 sec)

ERROR: 
No query specified


```
查看MySQL error日志,error log中报错如下：

```
2019-06-20 12:44:40 768 [Note] Server socket created on IP: '::'.
2019-06-20 12:44:41 768 [Warning] 'proxies_priv' entry '@ root@jst-mysql-db03' ignored in --skip-name-resolve mode.
2019-06-20 12:44:41 768 [Warning] Neither --relay-log nor --relay-log-index were used; so replication may break when this MySQL server acts as a slave and has his hostname changed!! Please use '--relay
-log=jst-101-centos3-relay-bin' to avoid this problem.
2019-06-20 12:44:42 768 [ERROR] Error in Log_event::read_log_event(): 'read error', data_len: 7771, event_type: 32
2019-06-20 12:44:42 768 [Warning] Error reading GTIDs from binary log: -1
2019-06-20 12:44:42 768 [Warning] Storing MySQL user name or password information in the master info repository is not secure and is therefore not recommended. Please consider using the USER and PASSWO
RD connection options for START SLAVE; see the 'START SLAVE Syntax' in the MySQL Manual for more information.
2019-06-20 12:44:42 768 [Warning] Slave SQL: If a crash happens this configuration does not guarantee that the relay log info will be consistent, Error_code: 0
2019-06-20 12:44:42 768 [Note] Slave SQL thread initialized, starting replication in log 'bin.000902' at position 817974423, relay log './jst-101-centos3-relay-bin.001800' position: 817974541
2019-06-20 12:44:42 768 [Note] Slave I/O thread: connected to master 'repl@10.10.202.178:3306',replication started in log 'bin.000906' at position 114393710
2019-06-20 12:44:42 768 [Note] Event Scheduler: Loaded 0 events
2019-06-20 12:44:42 768 [Note] /usr/local/mysql/bin/mysqld: ready for connections.
Version: '5.6.32-log'  socket: '/tmp/mysql.sock'  port: 3306  MySQL Community Server (GPL)
2019-06-20 12:57:03 768 [Warning] Disk is full writing './jst-101-centos3-relay-bin.001818' (Errcode: 28 - No space left on device). Waiting for someone to free space...
2019-06-20 12:57:03 768 [Warning] Retry in 60 secs. Message reprinted in 600 secs
2019-06-20 12:57:03 768 [ERROR] Slave SQL: Could not execute Delete_rows event on table jstpay.t_tran_balance_sett_book_park; Error writing file '/data/mysql/tmp/MLG6cLj1' (Errcode: 28 - No space left 
on device), Error_code: 3; Error writing file '/data/logs/bin' (errno: 28 - No space left on device), Error_code: 1026; handler error HA_ERR_RBR_LOGGING_FAILED; the event's master log bin.000902, end_l
og_pos 1826635584, Error_code: 3
2019-06-20 12:57:03 768 [Warning] Slave: Error writing file '/data/mysql/tmp/MLG6cLj1' (Errcode: 28 - No space left on device) Error_code: 3
2019-06-20 12:57:03 768 [Warning] Slave: Error writing file '/data/logs/bin' (errno: 28 - No space left on device) Error_code: 1026
2019-06-20 12:57:03 768 [ERROR] Error running query, slave SQL thread aborted. Fix the problem, and restart the slave SQL thread with "SLAVE START". We stopped at log 'bin.000902' position 817974423
2019-06-20 12:58:03 768 [Warning] Disk is full writing './jst-101-centos3-relay-bin.001818' (Errcode: 28 - No space left on device). Waiting for someone to free space...
2019-06-20 12:58:03 768 [Warning] Retry in 60 secs. Message reprinted in 600 secs
2019-06-20 13:08:03 768 [Warning] Disk is full writing './jst-101-centos3-relay-bin.001818' (Errcode: 28 - No space left on device). Waiting for someone to free space...
2019-06-20 13:08:03 768 [Warning] Retry in 60 secs. Message reprinted in 600 secs
2019-06-20 13:09:05 768 [ERROR] Error reading packet from server: Lost connection to MySQL server during query (server_errno=2013)
2019-06-20 13:09:05 768 [Note] Slave I/O thread: Failed reading log event, reconnecting to retry, log 'bin.000906' at position 1584047596
2019-06-20 13:09:05 768 [Warning] Storing MySQL user name or password information in the master info repository is not secure and is therefore not recommended. Please consider using the USER and PASSWO
RD connection options for START SLAVE; see the 'START SLAVE Syntax' in the MySQL Manual for more information.
2019-06-20 13:17:17 768 [Warning] Disk is full writing './jst-101-centos3-relay-bin.001819' (Errcode: 28 - No space left on device). Waiting for someone to free space...
2019-06-20 13:17:17 768 [Warning] Retry in 60 secs. Message reprinted in 600 secs
2019-06-20 13:18:17 768 [Warning] Disk is full writing './jst-101-centos3-relay-bin.001819' (Errcode: 28 - No space left on device). Waiting for someone to free space...
2019-06-20 13:18:17 768 [Warning] Retry in 60 secs. Message reprinted in 600 secs
2019-06-20 13:28:17 768 [Warning] Disk is full writing './jst-101-centos3-relay-bin.001819' (Errcode: 28 - No space left on device). Waiting for someone to free space...
2019-06-20 13:28:17 768 [Warning] Retry in 60 secs. Message reprinted in 600 secs
2019-06-20 13:38:17 768 [Warning] Disk is full writing './jst-101-centos3-relay-bin.001819' (Errcode: 28 - No space left on device). Waiting for someone to free space...
2019-06-20 13:38:17 768 [Warning] Retry in 60 secs. Message reprinted in 600 secs


```
## 3.分析问题

   从上面的报错信息中我们可以看到应该是主库的binlog日志或者从库的relay日志损坏，导致从库读取日志的时候时候，从而导致从库复制线程报错。

报错中也给出了检查主库binlog或者从库relay log是否损坏的方法，即通过mysqlbinlog解析binlog或是relay log看能否成功。

   检查发现mysqlbinlog能够正常解析主库binlog，但是在解析从库的relaylog时报如下错误：

```
mysqlbinlog  --base64-output=decode-rows -vvv  jst-101-centos3-relay-bin.001818 > ~/test.sql
#190620 11:05:44 server id 1783306  end_log_pos 114393710 CRC32 0x685f2cac 	Delete_rows: table id 583134
ERROR: Error in Log_event::read_log_event(): 'read error', data_len: 7771, event_type: 32
ERROR: Could not read entry at offset 114393828: Error in log format or read error.
WARNING: The range of printed events ends with a row event or a table map event that does not have the STMT_END_F flag set. This might be because the last statement was not fully written to the log, or because you are using a --stop-position or --stop-datetime that refers to an event in the middle of a statement. The event(s) from the partial statement have not been written to output.
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@jst-101-centos3 mysql]# ll

```
  到此我们可以确定是因为服务器断电导致从库的relay log损坏，从而导致从库复制线程报错。
  
  
## 4.解决方案

    relay log损坏，那么我们可以从复制停止的位置开始重新读取主库的binlog日志。有两点需要我们注意的1. 要确认要读取的binlog日志在主库还没有被删除；2. 确定正确的重新读取位置。

     我们要怎么确定读取位置？1. 可以通过show slave status\G;输出中的Relay_Master_Log_File，Exec_Master_Log_Pos来确定；2. 如果show slave status\G;中的值你搞不清楚，也可以通过从库errorlog中的 报错来确定位置。

    下面是我处理的具体步骤：

```

mysql> stop slave ;
Query OK, 0 rows affected (0.00 sec)
 
mysql> reset slave all;
Query OK, 0 rows affected (0.15 sec)
 
mysql> CHANGE MASTER TO  
     MASTER_HOST='10.101.222.138',    
     MASTER_USER='repl',    
     MASTER_PASSWORD='repl#pass',    
     MASTER_PORT=3306,    
     MASTER_AUTO_POSITION = 1;
Query OK, 0 rows affected, 2 warnings (0.05 sec)
 
mysql> start slave;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status\G;


*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.101.222.138
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: bin.000934
          Read_Master_Log_Pos: 900561948
               Relay_Log_File: jst-101-centos3-relay-bin.000086
                Relay_Log_Pos: 900562066
        Relay_Master_Log_File: bin.000934
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 900561948
              Relay_Log_Space: 900562361
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 3555150
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1783306
                  Master_UUID: 8448095b-2779-11e8-9eee-801844f2eb44
             Master_Info_File: /data/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 8448095b-2779-11e8-9eee-801844f2eb44:48404439-49078256
            Executed_Gtid_Set: 8448095b-2779-11e8-9eee-801844f2eb44:1-49078256,
b29adcee-d60a-11e6-8697-0050568c3815:1-151573105,
ec3d8ee4-da32-11e6-a1b2-0050568c769d:1-6
                Auto_Position: 1
1 row in set (0.02 sec)

ERROR: 
No query specified

mysql> 

```
