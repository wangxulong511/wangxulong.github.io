今天搭建MySQL使用upgrade后数据库报错无法启动
后删除了共享表空间，undo,redo log 后数据库可以启动  
报错提示查看MySQL官方文档
>https://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting-datadict.html


Key caches:
default
Buffer_size:       8388608
Block_size:           1024
Division_limit:        100
Age_limit:             300
blocks used:             7
not flushed:             0
w_requests:              2
writes:                  2
r_requests:             62
reads:                  10


handler status:
read_key:           59
read_next:          11
read_rnd             0
read_first:         45
write:            2937
delete               0
update:              1

Table status:
Opened tables:        419
Open tables:          265
Open files:            28
Open streams:           0

Memory status:
<malloc version="1">
<heap nr="0">
<sizes>
<unsorted from="0" to="0" total="0" count="139900695105464"/>
</sizes>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<system type="current" size="0"/>
<system type="max" size="0"/>
<aspace type="total" size="0"/>
<aspace type="mprotect" size="0"/>
</heap>
<total type="fast" count="0" size="0"/>
<total type="rest" count="0" size="0"/>
<total type="mmap" count="0" size="0"/>
<system type="current" size="0"/>
<system type="max" size="0"/>
<aspace type="total" size="0"/>
<aspace type="mprotect" size="0"/>
</malloc>



Events status:
LLA = Last Locked At  LUA = Last Unlocked At
WOC = Waiting On Condition  DL = Data Locked

Event scheduler status:
State      : RUNNING
Thread id  : 1
LLA        : is_running:637
LUA        : is_running:639
WOC        : NO
Workers    : 0
Executed   : 0
Data locked: NO

Event queue status:
Element count   : 0
Data locked     : NO
Attempting lock : NO
LLA             : get_top_for_execution_if_time:558
LUA             : get_top_for_execution_if_time:579
WOC             : YES
Next activation : never
2019-01-04T09:14:35.155707Z 0 [Warning] 'user' entry 'root@localhost' ignored in --skip-name-resolve mode.
2019-01-04T09:14:35.155774Z 0 [Warning] 'user' entry 'mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-01-04T09:14:35.155785Z 0 [Warning] 'user' entry 'mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-01-04T09:14:35.155829Z 0 [Warning] 'db' entry 'performance_schema mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-01-04T09:14:35.155837Z 0 [Warning] 'db' entry 'sys mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-01-04T09:14:35.155847Z 0 [Warning] 'proxies_priv' entry '@ root@localhost' ignored in --skip-name-resolve mode.
2019-01-04T09:14:35.156126Z 0 [Warning] 'tables_priv' entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-01-04T09:14:35.156142Z 0 [Warning] 'tables_priv' entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:43:26.224528Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T08:43:27.675911Z 0 [Warning] InnoDB: Resizing redo log from 2*65536 to 2*262144 pages, LSN=2512126
2019-03-20T08:43:27.780352Z 0 [Warning] InnoDB: Starting to delete and rewrite log files.
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000 2100 2200 2300 2400 2500 2600 2700 2800 2900 3000 3100 3200 3300 3400 3500 3600 3700 3800 3900 4000
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000 2100 2200 2300 2400 2500 2600 2700 2800 2900 3000 3100 3200 3300 3400 3500 3600 3700 3800 3900 4000
2019-03-20T08:43:36.047508Z 0 [Warning] InnoDB: New log files created, LSN=2512126
mysqld: File '/data/logs/binlog.000006' not found (Errcode: 2 - No such file or directory)
2019-03-20T08:43:36.138541Z 0 [ERROR] Failed to open log (file '/data/logs/binlog.000006', errno 2)
2019-03-20T08:43:36.138545Z 0 [ERROR] Could not open log file
2019-03-20T08:43:36.138549Z 0 [ERROR] Can't init tc log
2019-03-20T08:43:36.138551Z 0 [ERROR] Aborting

2019-03-20T08:46:20.803386Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T08:46:22.379023Z 0 [Warning] InnoDB: Table mysql/innodb_table_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:46:22.379060Z 0 [Warning] InnoDB: Table mysql/innodb_index_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:46:22.382893Z 0 [Warning] CA certificate ca.pem is self signed.
2019-03-20T08:46:22.391026Z 0 [Warning] 'user' entry 'root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:46:22.391057Z 0 [Warning] 'user' entry 'mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:46:22.391069Z 0 [Warning] 'user' entry 'mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:46:22.391086Z 0 [Warning] 'db' entry 'performance_schema mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:46:22.391090Z 0 [Warning] 'db' entry 'sys mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:46:22.391098Z 0 [Warning] 'proxies_priv' entry '@ root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:46:22.398604Z 0 [Warning] 'tables_priv' entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:46:22.398615Z 0 [Warning] 'tables_priv' entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:09.661612Z 4 [Warning] InnoDB: Table mysql/innodb_table_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:47:09.661629Z 4 [Warning] InnoDB: Table mysql/innodb_index_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:47:10.048935Z 4 [Warning] 'user' entry 'root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.048955Z 4 [Warning] 'user' entry 'mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.048962Z 4 [Warning] 'user' entry 'mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.048977Z 4 [Warning] 'db' entry 'performance_schema mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.048981Z 4 [Warning] 'db' entry 'sys mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.048987Z 4 [Warning] 'proxies_priv' entry '@ root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.049014Z 4 [Warning] 'tables_priv' entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.049021Z 4 [Warning] 'tables_priv' entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.119712Z 4 [Warning] InnoDB: Table mysql/innodb_table_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:47:10.119724Z 4 [Warning] InnoDB: Table mysql/innodb_index_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:47:10.119752Z 4 [Warning] InnoDB: Table mysql/innodb_table_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:47:10.119759Z 4 [Warning] InnoDB: Table mysql/innodb_index_stats has length mismatch in the column name table_name.  Please run mysql_upgrade
2019-03-20T08:47:10.684847Z 4 [Warning] 'user' entry 'root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.684869Z 4 [Warning] 'user' entry 'mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.684876Z 4 [Warning] 'user' entry 'mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.684890Z 4 [Warning] 'db' entry 'performance_schema mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.684894Z 4 [Warning] 'db' entry 'sys mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.684901Z 4 [Warning] 'proxies_priv' entry '@ root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.684924Z 4 [Warning] 'tables_priv' entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:47:10.684931Z 4 [Warning] 'tables_priv' entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:36.531623Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T08:48:38.088084Z 0 [Warning] CA certificate ca.pem is self signed.
2019-03-20T08:48:38.092848Z 0 [Warning] 'user' entry 'root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:38.092885Z 0 [Warning] 'user' entry 'mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:38.092899Z 0 [Warning] 'user' entry 'mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:38.092914Z 0 [Warning] 'db' entry 'performance_schema mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:38.092921Z 0 [Warning] 'db' entry 'sys mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:38.092928Z 0 [Warning] 'proxies_priv' entry '@ root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:38.101726Z 0 [Warning] 'tables_priv' entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T08:48:38.101743Z 0 [Warning] 'tables_priv' entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T09:34:26.564332Z 6 [Warning] Timeout waiting for reply of binlog (file: binlog.000002, pos: 398), semi-sync up to file , position 0.
2019-03-20T10:48:40.571311Z 11 [ERROR] InnoDB: Flag mismatch in page [page id: space=38, page number=3] index `PRIMARY` of table `mysql`.`help_relation`
2019-03-20 18:48:40 0x7fa42c3b1700  InnoDB: Assertion failure in thread 140343093434112 in file btr0btr.cc line 173
InnoDB: We intentionally generate a memory trap.
InnoDB: Submit a detailed bug report to http://bugs.mysql.com.
InnoDB: If you get repeated assertion failures or crashes, even
InnoDB: immediately after the mysqld startup, there may be
InnoDB: corruption in the InnoDB tablespace. Please refer to
InnoDB: http://dev.mysql.com/doc/refman/5.7/en/forcing-innodb-recovery.html
InnoDB: about forcing recovery.
10:48:40 UTC - mysqld got signal 6 ;
This could be because you hit a bug. It is also possible that this binary
or one of the libraries it was linked against is corrupt, improperly built,
or misconfigured. This error can also be caused by malfunctioning hardware.
Attempting to collect some information that could help diagnose the problem.
As this is a crash and something is definitely wrong, the information
collection process might fail.

key_buffer_size=8388608
read_buffer_size=16777216
max_used_connections=1
max_threads=5000
thread_count=2
connection_count=1
It is possible that mysqld could use up to 
key_buffer_size + (read_buffer_size + sort_buffer_size)*max_threads = 245835067 K  bytes of memory
Hope that's ok; if not, decrease some variables in the equation.

Thread pointer: 0x7fa3e0012550
Attempting backtrace. You can use the following information to find out
where mysqld died. If you see no messages after this, something went
terribly wrong...
stack_bottom = 7fa42c3b0e28 thread_stack 0x40000
/usr/local/mysql/bin/mysqld(my_print_stacktrace+0x35)[0xf4e5e5]
/usr/local/mysql/bin/mysqld(handle_fatal_signal+0x4a4)[0x7d1b64]
/lib64/libpthread.so.0[0x3ff240f7e0]
/lib64/libc.so.6(gsignal+0x35)[0x3ff20325e5]
/lib64/libc.so.6(abort+0x175)[0x3ff2033dc5]
/usr/local/mysql/bin/mysqld(_Z18ut_print_timestampP8_IO_FILE+0x0)[0x7c0f4e]
/usr/local/mysql/bin/mysqld(_Z18btr_root_block_getPK12dict_index_tmP5mtr_t+0x1bb)[0x11b3d3b]
/usr/local/mysql/bin/mysqld(_Z12btr_root_getPK12dict_index_tP5mtr_t+0x11)[0x11b3d71]
/usr/local/mysql/bin/mysqld(_Z12btr_get_sizeP12dict_index_tmP5mtr_t+0x34)[0x11b3e84]
/usr/local/mysql/bin/mysqld[0x123d179]
/usr/local/mysql/bin/mysqld(_Z27dict_stats_update_transientP12dict_table_t+0x181)[0x123d5f1]
/usr/local/mysql/bin/mysqld(_Z17dict_stats_updateP12dict_table_t23dict_stats_upd_option_t+0x69)[0x1240919]
/usr/local/mysql/bin/mysqld(_ZN11ha_innobase4openEPKcij+0x3ac)[0x104749c]
/usr/local/mysql/bin/mysqld(_ZN7handler7ha_openEP5TABLEPKcii+0x3e)[0x81c5ce]
/usr/local/mysql/bin/mysqld(_Z21open_table_from_shareP3THDP11TABLE_SHAREPKcjjjP5TABLEb+0x796)[0xdc4d66]
/usr/local/mysql/bin/mysqld(_Z10open_tableP3THDP10TABLE_LISTP18Open_table_context+0xf9)[0xcc2ae9]
/usr/local/mysql/bin/mysqld(_Z11open_tablesP3THDPP10TABLE_LISTPjjP19Prelocking_strategy+0xd76)[0xcc64e6]
/usr/local/mysql/bin/mysqld(_Z21open_tables_for_queryP3THDP10TABLE_LISTj+0x52)[0xcc70f2]
/usr/local/mysql/bin/mysqld[0xd62fc4]
/usr/local/mysql/bin/mysqld(_Z14get_all_tablesP3THDP10TABLE_LISTP4Item+0x6eb)[0xd70e2b]
/usr/local/mysql/bin/mysqld[0xd5eb58]
/usr/local/mysql/bin/mysqld(_Z24get_schema_tables_resultP4JOIN23enum_schema_table_state+0x1a9)[0xd5edb9]
/usr/local/mysql/bin/mysqld(_ZN4JOIN14prepare_resultEv+0x6d)[0xd5445d]
/usr/local/mysql/bin/mysqld(_ZN4JOIN4execEv+0xc0)[0xceb2d0]
/usr/local/mysql/bin/mysqld(_Z12handle_queryP3THDP3LEXP12Query_resultyy+0x250)[0xd55820]
/usr/local/mysql/bin/mysqld[0xd16463]
/usr/local/mysql/bin/mysqld(_Z21mysql_execute_commandP3THDb+0x3798)[0xd19ec8]
/usr/local/mysql/bin/mysqld(_Z11mysql_parseP3THDP12Parser_state+0x40d)[0xd1b97d]
/usr/local/mysql/bin/mysqld(_Z16dispatch_commandP3THDPK8COM_DATA19enum_server_command+0x11a5)[0xd1cba5]
/usr/local/mysql/bin/mysqld(_Z10do_commandP3THD+0x194)[0xd1da54]
/usr/local/mysql/bin/mysqld(handle_connection+0x29c)[0xdef3fc]
/usr/local/mysql/bin/mysqld(pfs_spawn_thread+0x174)[0xfc79a4]
/lib64/libpthread.so.0[0x3ff2407aa1]
/lib64/libc.so.6(clone+0x6d)[0x3ff20e8aad]

Trying to get some variables.
Some pointers may be invalid and cause the dump to abort.
Query (7fa3e0022a40): is an invalid pointer
Connection ID (thread ID): 11
Status: NOT_KILLED

The manual page at http://dev.mysql.com/doc/mysql/en/crashing.html contains
information that should help you find out what is causing the crash.
2019-03-20T10:48:40.898554Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T10:48:42.011226Z 0 [ERROR] InnoDB: Header page consists of zero bytes in datafile: ./ibdata1, Space ID:0, Flags: 0. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting-datadict.html for how to resolve the issue.
2019-03-20T10:48:42.011273Z 0 [ERROR] InnoDB: Corrupted page [page id: space=0, page number=0] of datafile './ibdata1' could not be found in the doublewrite buffer.
2019-03-20T10:48:42.011283Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Data structure corruption
2019-03-20T10:48:42.611917Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2019-03-20T10:48:42.611928Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2019-03-20T10:48:42.611934Z 0 [ERROR] Failed to initialize builtin plugins.
2019-03-20T10:48:42.611938Z 0 [ERROR] Aborting

2019-03-20T10:49:28.624202Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T10:49:29.724952Z 0 [ERROR] InnoDB: Header page consists of zero bytes in datafile: ./ibdata1, Space ID:0, Flags: 0. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting-datadict.html for how to resolve the issue.
2019-03-20T10:49:29.724999Z 0 [ERROR] InnoDB: Corrupted page [page id: space=0, page number=0] of datafile './ibdata1' could not be found in the doublewrite buffer.
2019-03-20T10:49:29.725007Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Data structure corruption
2019-03-20T10:49:30.325635Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2019-03-20T10:49:30.325646Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2019-03-20T10:49:30.325653Z 0 [ERROR] Failed to initialize builtin plugins.
2019-03-20T10:49:30.325657Z 0 [ERROR] Aborting

2019-03-20T10:52:53.404830Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T10:52:54.511101Z 0 [ERROR] InnoDB: Header page consists of zero bytes in datafile: ./ibdata1, Space ID:0, Flags: 0. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting-datadict.html for how to resolve the issue.
2019-03-20T10:52:54.511150Z 0 [ERROR] InnoDB: Corrupted page [page id: space=0, page number=0] of datafile './ibdata1' could not be found in the doublewrite buffer.
2019-03-20T10:52:54.511171Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Data structure corruption
2019-03-20T10:52:55.111862Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2019-03-20T10:52:55.111886Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2019-03-20T10:52:55.111899Z 0 [ERROR] Failed to initialize builtin plugins.
2019-03-20T10:52:55.111906Z 0 [ERROR] Aborting

2019-03-20T10:54:15.587655Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T10:54:16.692409Z 0 [ERROR] InnoDB: undo tablespace './/undo001' exists. Creating system tablespace with existing undo tablespaces is not supported. Please delete all undo tablespaces before creating new system tablespace.
2019-03-20T10:54:16.692433Z 0 [ERROR] InnoDB: InnoDB Database creation was aborted with error Generic error. You may need to delete the ibdata1 file before trying to start up again.
2019-03-20T10:54:17.293062Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2019-03-20T10:54:17.293073Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2019-03-20T10:54:17.293080Z 0 [ERROR] Failed to initialize builtin plugins.
2019-03-20T10:54:17.293084Z 0 [ERROR] Aborting

2019-03-20T10:55:06.244876Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2019-03-20T10:55:07.353029Z 0 [ERROR] InnoDB: redo log file './ib_logfile0' exists. Creating system tablespace with existing redo log files is not recommended. Please delete all redo log files before creating new system tablespace.
2019-03-20T10:55:07.353053Z 0 [ERROR] InnoDB: InnoDB Database creation was aborted with error Generic error. You may need to delete the ibdata1 file before trying to start up again.
2019-03-20T10:55:07.953605Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2019-03-20T10:55:07.953631Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2019-03-20T10:55:07.953656Z 0 [ERROR] Failed to initialize builtin plugins.
2019-03-20T10:55:07.953659Z 0 [ERROR] Aborting

2019-03-20T10:56:04.753680Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000 2100 2200 2300 2400 2500 2600 2700 2800 2900 3000 3100 3200 3300 3400 3500 3600 3700 3800 3900 4000
 100 200 300 400 500 600 700 800 900 1000 1100 1200 1300 1400 1500 1600 1700 1800 1900 2000 2100 2200 2300 2400 2500 2600 2700 2800 2900 3000 3100 3200 3300 3400 3500 3600 3700 3800 3900 4000
2019-03-20T10:56:13.895056Z 0 [Warning] InnoDB: New log files created, LSN=48433
2019-03-20T10:56:13.931214Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2019-03-20T10:56:14.001240Z 0 [Warning] InnoDB: Cannot open table mysql/plugin from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
mysqld: Table 'mysql.plugin' doesn't exist
2019-03-20T10:56:14.001274Z 0 [ERROR] Can't open the mysql.plugin table. Please run mysql_upgrade to create it.
2019-03-20T10:56:14.007299Z 0 [Warning] InnoDB: Cannot open table mysql/gtid_executed from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
mysqld: Table 'mysql.gtid_executed' doesn't exist
2019-03-20T10:56:14.007321Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2019-03-20T10:56:14.009063Z 0 [Warning] CA certificate ca.pem is self signed.
2019-03-20T10:56:14.018449Z 0 [Warning] InnoDB: Cannot open table mysql/server_cost from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
2019-03-20T10:56:14.018477Z 0 [Warning] Failed to open optimizer cost constant tables

2019-03-20T10:56:14.018810Z 0 [Warning] 'user' entry 'root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.018841Z 0 [Warning] 'user' entry 'mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.018848Z 0 [Warning] 'user' entry 'mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.018867Z 0 [Warning] 'db' entry 'performance_schema mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.018871Z 0 [Warning] 'db' entry 'sys mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.018878Z 0 [Warning] 'proxies_priv' entry '@ root@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.018960Z 0 [Warning] InnoDB: Cannot open table mysql/time_zone_leap_second from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
2019-03-20T10:56:14.018973Z 0 [Warning] Can't open and lock time zone table: Table 'mysql.time_zone_leap_second' doesn't exist trying to live without them
2019-03-20T10:56:14.019196Z 0 [Warning] 'tables_priv' entry 'user mysql.session@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.019209Z 0 [Warning] 'tables_priv' entry 'sys_config mysql.sys@localhost' ignored in --skip-name-resolve mode.
2019-03-20T10:56:14.019289Z 0 [Warning] InnoDB: Cannot open table mysql/servers from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
2019-03-20T10:56:14.019301Z 0 [ERROR] Can't open and lock privilege tables: Table 'mysql.servers' doesn't exist
2019-03-20T10:56:14.019517Z 0 [Warning] InnoDB: Cannot open table mysql/slave_master_info from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
2019-03-20T10:56:14.019598Z 0 [Warning] InnoDB: Cannot open table mysql/slave_relay_log_info from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
2019-03-20T10:56:14.019663Z 0 [Warning] InnoDB: Cannot open table mysql/slave_master_info from the internal data dictionary of InnoDB though the .frm file for the table exists. Please refer to http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html for how to resolve the issue.
2019-03-20T10:56:14.019673Z 0 [Warning] Info table is not ready to be used. Table 'mysql.slave_master_info' cannot be opened.
2019-03-20T10:56:14.019681Z 0 [ERROR] Error in checking mysql.slave_master_info repository info type of TABLE.
2019-03-20T10:56:14.019686Z 0 [ERROR] Error creating master info: Error checking repositories.
2019-03-20T10:56:14.019688Z 0 [ERROR] Slave: Failed to initialize the master info structure for channel ''; its record may still be present in 'mysql.slave_master_info' table, consider deleting it.
2019-03-20T10:56:14.019692Z 0 [ERROR] Failed to create or recover replication info repositories.
