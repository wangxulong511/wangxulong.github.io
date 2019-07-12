---
author: "wangxulong511"
description: "Oracle 11gR2 DG部署（RMAN方式）"
date: "2019-07-12"
categories: ["DataBase", "DB"]
tags: ["oracle DG"]
type: "post"
---  


# Oracle 11gR2 DG部署（RMAN方式）

### Oracle 11gR2 DG部署（RMAN方式）
> https://blog.csdn.net/onlyshenmin/article/details/81069263#51DG_switchover__317    

###　Oracle 11G DG之Duplicate方式搭建
> http://blog.itpub.net/28569596/viewspace-2150303/

## 搭建DG主库执行的脚本
```
alter system set db_unique_name='testdb' scope=spfile;
alter system set log_archive_config='dg_config=(testdb,dytkdg)' scope=spfile;
alter system set log_archive_dest_1='location=/archivelog VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=testdb' scope=spfile;
alter system set log_archive_dest_2='service=dytkdg LGWR SYNC  AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=dytkdg' scope=spfile;
alter system set log_archive_dest_state_1=ENABLE scope=spfile;
alter system set log_archive_dest_state_2=ENABLE scope=spfile;
alter system set remote_login_passwordfile=EXCLUSIVE scope=spfile;
alter system set fal_server='dytkdg' scope=spfile;
alter system set fal_client='testdb' scope=spfile;
alter system set db_file_name_convert ='/data/testdb','/data/testdb' scope=spfile;
alter system set log_file_name_convert='/data/testdb','/data/testdb' scope=spfile;
alter system set standby_file_management='AUTO' scope=spfile;

```
通过spfile 创建pfile 
> create pfile='/home/oracle/pfile.ora' from spfile;

修改pfile并发送到备库,修改后的备库pfile文件

```
testdb.__db_cache_size=1476395008
testdb.__java_pool_size=16777216
testdb.__large_pool_size=33554432
testdb.__oracle_base='/u01/app/oracle'#ORACLE_BASE set from environment
testdb.__pga_aggregate_target=1056964608
testdb.__sga_target=1996488704
testdb.__shared_io_pool_size=0
testdb.__shared_pool_size=452984832
testdb.__streams_pool_size=0
*.audit_file_dest='/u01/app/oracle/admin/dytkdg/adump'
*.audit_sys_operations=TRUE
*.audit_trail='FALSE'
*.compatible='11.2.0.4.0'
*.control_files='/data/testdb/control01.ctl','/u01/app/oracle/fast_recovery_area/dytkdg/control02.ctl'
*.db_block_size=8192
*.db_domain=''
*.db_file_name_convert='/data/testdb','/data/testdb'
*.db_name='testdb'
*.db_recovery_file_dest='/u01/app/oracle/fast_recovery_area'
*.db_recovery_file_dest_size=4385144832
*.db_unique_name='dytkdg'
*.diagnostic_dest='/u01/app/oracle'
*.dispatchers='(PROTOCOL=TCP) (SERVICE=testdbXDB)'
*.fal_client='dytkdg'
*.fal_server='testdb'
*.log_archive_config='dg_config=(testdb,dytkdg)'
*.log_archive_dest_1='location=/archivelog VALID_FOR=(ALL_LOGFILES,ALL_ROLES) DB_UNIQUE_NAME=dytkdg'
*.log_archive_dest_2='service=testdb LGWR SYNC  AFFIRM VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=testdb'
*.log_archive_dest_state_1='ENABLE'
*.log_archive_dest_state_2='ENABLE'
*.log_archive_format='arch_%t_%s_%r.arc'
*.log_file_name_convert='/data/testdb','/data/testdb'
*.memory_max_target=3040870400
*.memory_target=3040870400
*.open_cursors=2000
*.pga_aggregate_target=0
*.pre_page_sga=FALSE
*.processes=2000
*.remote_login_passwordfile='EXCLUSIVE'
*.sessions=2000
*.sga_max_size=2128609280
*.sga_target=0
*.standby_file_management='AUTO'
*.undo_tablespace='UNDOTBS1'

```

启动备库 startup nomonut

>SQL> startup nomoun

rman DUPLICATE 

```
rman target sys/SIGasmlib@HSIDBPR auxiliary sys/SIGasmlib@HSIDBSD 
RMAN>duplicate target database for standby from active database nofilenamecheck dorecover;
备库启动到open状态.


SQL> alter database recover managed standby database using current logfile disconnect from session;
 

Database altered.

SQL> archive log list;
Database log mode	       Archive Mode
Automatic archival	       Enabled
Archive destination	       /archivelog
Oldest online log sequence     0
Next log sequence to archive   0
Current log sequence	       0
SQL> select unique(thread#),max(sequence#) over(partition by thread#) from v$archived_log;

   THREAD# MAX(SEQUENCE#)OVER(PARTITIONBYTHREAD#)
---------- --------------------------------------
	 1				     1265

SQL> alter database set standby database to maximize availability;

Database altered.

SQL> alter database open;
alter database open
*
ERROR at line 1:
ORA-10456: cannot open standby database; media recovery session may be in
progress


SQL> shutdown immediate
ORA-01109: database not open


Database dismounted.
ORACLE instance shut down.
SQL> startup
ORACLE instance started.

Total System Global Area 2121183232 bytes
Fixed Size		    2254904 bytes
Variable Size		 1526728648 bytes
Database Buffers	  587202560 bytes
Redo Buffers		    4997120 bytes
Database mounted.
Database opened.
SQL> select database_role,protection_mode,protection_level,open_mode from v$database;

DATABASE_ROLE			 PROTECTION_MODE
-------------------------------- ----------------------------------------
PROTECTION_LEVEL
----------------------------------------
OPEN_MODE
----------------------------------------
PHYSICAL STANDBY		 MAXIMUM PERFORMANCE
MAXIMUM PERFORMANCE
READ ONLY


SQL> select process,client_process,sequence#,status from v$managed_standby;

PROCESS 	   CLIENT_PROCESS    SEQUENCE# STATUS
------------------ ---------------- ---------- ------------------------
ARCH		   ARCH 		     0 CONNECTED
ARCH		   ARCH 		  1268 CLOSING
ARCH		   ARCH 		     0 CONNECTED
ARCH		   ARCH 		  1269 CLOSING
RFS		   ARCH 		     0 IDLE
RFS		   UNKNOWN		     0 IDLE
RFS		   LGWR 		  1270 IDLE

7 rows selected.

```

##　oracle　DG 主备切换 

查看主库
```
select switchover_status,database_role,open_mode from v$database; 
SWITCHOVER_STATUS    DATABASE_ROLE    OPEN_MODE
-------------------- ---------------- --------------------
SESSIONS ACTIVE      PRIMARY	      READ WRITE
```
主库切换备库 
```
alter database commit to switchover to physical standby with session shutdown;

```
查看主库日志,通过日志可以看出主库切换为备库后会关闭实例
tail -333f /u01/app/oracle/diag/rdbms/testdb/testdb/trace/alert_testdb.log
```
Fri Jul 12 14:02:46 2019
Archived Log entry 1272 added for thread 1 sequence 1271 ID 0xa363b55b dest 1:
Fri Jul 12 14:03:08 2019
ALTER SYSTEM SET log_archive_dest_state_2='ENABLE' SCOPE=MEMORY SID='*';
Fri Jul 12 14:03:08 2019
ARC0: Standby redo logfile selected for thread 1 sequence 1271 for destination LOG_ARCHIVE_DEST_2
Fri Jul 12 14:03:09 2019
Thread 1 cannot allocate new log, sequence 1273
Checkpoint not complete
  Current log# 1 seq# 1272 mem# 0: /data/testdb/redo01.log
Destination LOG_ARCHIVE_DEST_2 is SYNCHRONIZED
LGWR: Standby redo logfile selected for thread 1 sequence 1273 for destination LOG_ARCHIVE_DEST_2
Thread 1 advanced to log sequence 1273 (LGWR switch)
  Current log# 3 seq# 1273 mem# 0: /data/testdb/redo03.log
Fri Jul 12 14:03:13 2019
Archived Log entry 1274 added for thread 1 sequence 1272 ID 0xa363b55b dest 1:
ARC0: Standby redo logfile selected for thread 1 sequence 1272 for destination LOG_ARCHIVE_DEST_2
Fri Jul 12 14:04:05 2019
alter database commit to switchover to physical standby with session shutdown
ALTER DATABASE COMMIT TO SWITCHOVER TO PHYSICAL STANDBY [Process Id: 13518] (testdb)
Waiting for all non-current ORLs to be archived...
All non-current ORLs have been archived.
Waiting for all FAL entries to be archived...
All FAL entries have been archived.
Waiting for potential Physical Standby switchover target to become synchronized...
Active, synchronized Physical Standby switchover target has been identified
Switchover End-Of-Redo Log thread 1 sequence 1273 has been fixed
Switchover: Primary highest seen SCN set to 0x0.0xc74fcbea
ARCH: Noswitch archival of thread 1, sequence 1273
ARCH: End-Of-Redo Branch archival of thread 1 sequence 1273
ARCH: LGWR is actively archiving destination LOG_ARCHIVE_DEST_2
ARCH: Standby redo logfile selected for thread 1 sequence 1273 for destination LOG_ARCHIVE_DEST_2
Archived Log entry 1276 added for thread 1 sequence 1273 ID 0xa363b55b dest 1:
ARCH: Archiving is disabled due to current logfile archival
Primary will check for some target standby to have received alls redo
Final check for a synchronized target standby. Check will be made once.
LOG_ARCHIVE_DEST_2 is a potential Physical Standby switchover target
Active, synchronized target has been identified
Target has also received all redo
Backup controlfile written to trace file /u01/app/oracle/diag/rdbms/testdb/testdb/trace/testdb_ora_13518.trc
Clearing standby activation ID 2741220699 (0xa363b55b)
The primary database controlfile was created using the
'MAXLOGFILES 16' clause.
There is space for up to 13 standby redo logfiles
Use the following SQL commands on the standby database to create
standby redo logfiles that match the primary database:
ALTER DATABASE ADD STANDBY LOGFILE 'srl1.f' SIZE 157286400;
ALTER DATABASE ADD STANDBY LOGFILE 'srl2.f' SIZE 157286400;
ALTER DATABASE ADD STANDBY LOGFILE 'srl3.f' SIZE 157286400;
ALTER DATABASE ADD STANDBY LOGFILE 'srl4.f' SIZE 157286400;
Archivelog for thread 1 sequence 1273 required for standby recovery
Switchover: Primary controlfile converted to standby controlfile succesfully.
Switchover: Complete - Database shutdown required
USER (ospid: 13518): terminating the instance
Instance terminated by USER, pid = 13518
Completed: alter database commit to switchover to physical standby with session shutdown
Shutting down instance (abort)
License high water mark = 137
Fri Jul 12 14:04:06 2019
Instance shutdown complete
Fri Jul 12 14:11:38 2019
```

