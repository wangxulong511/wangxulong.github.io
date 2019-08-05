
# Greenplum 5.16.0初探
> https://blog.51cto.com/lee90/2371858?source=drt

##  centos7.3下 greenplum-db 安装、配置文档
> https://segmentfault.com/a/1190000013001945

## centos6.9 greenplum5.3离线部署文档
> https://www.cnblogs.com/chenminklutz/p/9019831.html

## greenplum 常用客户端工具
> https://blog.csdn.net/xfg0218/article/details/90412285

## greenplum用户权限管理
> https://blog.csdn.net/weixin_42658788/article/details/88040209

## 客户端如何访问greenplum
> https://blog.csdn.net/kwame211/article/details/76229416

## python连接Greenplum数据库
> https://www.cnblogs.com/xiao-apple36/p/10362367.html

> greenplum 
https://www.cnblogs.com/ctypyb2002/category/1319670.html
## 异常处理 
##　Greenplum初始化数据库时出现gpadmin-[ERROR]:-gpstart error: Do not have enough valid segments to start the arr
> https://blog.csdn.net/double_happiness/article/details/83142299

centos7.4 安装 5.20
4C8G100G

```
[gpadmin@dw-greenplum-1 ~]$ cd gpconfigs/
[gpadmin@dw-greenplum-1 gpconfigs]$ gpssh-exkeys -f hostlist
[STEP 1 of 5] create local ID and authorize on local host
  ... /home/gpadmin/.ssh/id_rsa file exists ... key generation skipped

[STEP 2 of 5] keyscan all hosts and update known_hosts file

[STEP 3 of 5] authorize current user on remote hosts
  ... send to sdw1
  ... send to sdw2

[STEP 4 of 5] determine common authentication file content

[STEP 5 of 5] copy authentication files to all remote hosts
  ... finished key exchange with sdw1
  ... finished key exchange with sdw2

[INFO] completed successfully
[gpadmin@dw-greenplum-1 gpconfigs]$ gpssh -f hostlist -e date
[sdw1] date
[sdw1] Wed Jul 31 13:40:08 CST 2019
[ mdw] date
[ mdw] Wed Jul 31 13:40:08 CST 2019
[sdw2] date
[sdw2] Wed Jul 31 13:40:08 CST 2019
[gpadmin@dw-greenplum-1 gpconfigs]$ cp /usr/local/greenplum-db/docs/cli_help/gpconfigs/gpinitsystem_config .
[gpadmin@dw-greenplum-1 gpconfigs]$ ll
total 12
-rwxr-xr-x. 1 gpadmin gpadmin 2663 Jul 31 13:40 gpinitsystem_config
-rw-r--r--. 1 gpadmin gpadmin   14 Jul 31  2019 hostlist
-rw-r--r--. 1 gpadmin gpadmin   10 Jul 31  2019 seglist
[gpadmin@dw-greenplum-1 gpconfigs]$ vim gpinitsystem_config 
[gpadmin@dw-greenplum-1 gpconfigs]$ ll
total 12
-rwxr-xr-x. 1 gpadmin gpadmin 2901 Jul 31 13:46 gpinitsystem_config
-rw-r--r--. 1 gpadmin gpadmin   14 Jul 31  2019 hostlist
-rw-r--r--. 1 gpadmin gpadmin   10 Jul 31  2019 seglist
[gpadmin@dw-greenplum-1 gpconfigs]$ gpinitsystem -c gpinitsystem_config -h seglist -n zh_CN.utf8
20190731:13:47:31:010473 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking configuration parameters, please wait...
20190731:13:47:31:010473 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Reading Greenplum configuration file gpinitsystem_config
20190731:13:47:31:010473 gpinitsystem:dw-greenplum-1:gpadmin-[WARN]:-Master hostname mdw does not match hostname output
20190731:13:47:31:010473 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking to see if mdw can be resolved on this host
20190731:13:47:32:010473 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Can resolve mdw to this host
/bin/touch: cannot touch ‘/data/gpdata/master/tmp_file_test’: No such file or directory
20190731:13:47:32:gpinitsystem:dw-greenplum-1:gpadmin-[FATAL]:-Cannot write to /data/gpdata/master on master host  Script Exiting!
[gpadmin@dw-greenplum-1 gpconfigs]$ gpssh -f hostlist -e 'mkdir -p /data/gpdata/master'
[sdw1] mkdir -p /data/gpdata/master
[ mdw] mkdir -p /data/gpdata/master
[sdw2] mkdir -p /data/gpdata/master
[gpadmin@dw-greenplum-1 gpconfigs]$ gpssh -f hostlist -e 'mkdir -p /data/gpdata/primary'
[sdw1] mkdir -p /data/gpdata/primary
[ mdw] mkdir -p /data/gpdata/primary
[sdw2] mkdir -p /data/gpdata/primary
[gpadmin@dw-greenplum-1 gpconfigs]$ gpssh -f hostlist -e 'mkdir -p /data/gpdata/mirror'
[sdw1] mkdir -p /data/gpdata/mirror
[sdw2] mkdir -p /data/gpdata/mirror
[ mdw] mkdir -p /data/gpdata/mirror
[gpadmin@dw-greenplum-1 gpconfigs]$ gpinitsystem -c gpinitsystem_config -h seglist -n zh_CN.utf8
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking configuration parameters, please wait...
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Reading Greenplum configuration file gpinitsystem_config
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[WARN]:-Master hostname mdw does not match hostname output
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking to see if mdw can be resolved on this host
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Can resolve mdw to this host
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-No DATABASE_NAME set, will exit following template1 updates
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-MASTER_MAX_CONNECT not set, will set to default value 250
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking configuration parameters, Completed
20190731:13:49:57:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Commencing multi-home checks, please wait...
..
20190731:13:49:58:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Configuring build for standard array
20190731:13:49:58:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Commencing multi-home checks, Completed
20190731:13:49:58:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Building primary segment instance array, please wait...
....
20190731:13:50:01:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Building group mirror array type , please wait...
....
20190731:13:50:03:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking Master host
20190731:13:50:03:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking new segment hosts, please wait...
........
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checking new segment hosts, Completed
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Database Creation Parameters
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:---------------------------------------
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master Configuration
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:---------------------------------------
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master instance name       = Greenplum Data Platform
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master hostname            = mdw
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master port                = 5432
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master instance dir        = /data/gpdata/master/gpseg-1
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master LOCALE              = zh_CN.utf8
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Greenplum segment prefix   = gpseg
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master Database            = 
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master connections         = 250
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master buffers             = 128000kB
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Segment connections        = 750
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Segment buffers            = 128000kB
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Checkpoint segments        = 8
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Encoding                   = UNICODE
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Postgres param file        = Off
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Initdb to be used          = /usr/local/greenplum-db/./bin/initdb
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-GP_LIBRARY_PATH is         = /usr/local/greenplum-db/./lib
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-HEAP_CHECKSUM is           = on
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-HBA_HOSTNAMES is           = 0
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Ulimit check               = Passed
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Array host connect type    = Single hostname per node
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master IP address [1]      = ::1
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master IP address [2]      = 10.101.130.191
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Master IP address [3]      = fe80::2738:e7a9:9e39:5952
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Standby Master             = Not Configured
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Primary segment #          = 2
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Total Database segments    = 4
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Trusted shell              = ssh
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Number segment hosts       = 2
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Mirror port base           = 50000
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Replicaton port base       = 41000
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Mirror replicaton port base= 51000
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Mirror segment #           = 2
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Mirroring config           = ON
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Mirroring type             = Group
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:----------------------------------------
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Primary Segment Configuration
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:----------------------------------------
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw1 	/data/gpdata/primary/gpseg0 	6000 	2 	0 	41000
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw1 	/data/gpdata/primary/gpseg1 	6001 	3 	1 	41001
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw2 	/data/gpdata/primary/gpseg2 	6000 	4 	2 	41000
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw2 	/data/gpdata/primary/gpseg3 	6001 	5 	3 	41001
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:---------------------------------------
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Mirror Segment Configuration
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:---------------------------------------
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw2 	/data/gpdata/mirror/gpseg0 	50000 	6 	0 	51000
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw2 	/data/gpdata/mirror/gpseg1 	50001 	7 	1 	51001
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw1 	/data/gpdata/mirror/gpseg2 	50000 	8 	2 	51000
20190731:13:50:15:010976 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-sdw1 	/data/gpdata/mirror/gpseg3 	50001 	9 	3 	51001

Continue with Greenplum creation Yy|Nn (default=N):
> 
> y
20190731:13:52:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Building the Master instance database, please wait...
20190731:13:53:04:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Starting the Master in admin mode
20190731:13:53:10:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Commencing parallel build of primary segment instances
20190731:13:53:10:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Spawning parallel processes    batch [1], please wait...
....
20190731:13:53:10:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Waiting for parallel processes batch [1], please wait...
.................
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:------------------------------------------------
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Parallel process exit status
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:------------------------------------------------
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Total processes marked as completed           = 4
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Total processes marked as killed              = 0
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Total processes marked as failed              = 0
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:------------------------------------------------
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Commencing parallel build of mirror segment instances
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Spawning parallel processes    batch [1], please wait...
....
20190731:13:53:27:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Waiting for parallel processes batch [1], please wait...
................................
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:------------------------------------------------
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Parallel process exit status
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:------------------------------------------------
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Total processes marked as completed           = 4
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Total processes marked as killed              = 0
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Total processes marked as failed              = 0
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:------------------------------------------------
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Deleting distributed backout files
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Removing back out file
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-No errors generated from parallel processes
20190731:13:53:59:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Restarting the Greenplum instance in production mode
20190731:13:53:59:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Starting gpstop with args: -a -l /home/gpadmin/gpAdminLogs -i -m -d /data/gpdata/master/gpseg-1
20190731:13:53:59:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Gathering information and validating the environment...
20190731:13:53:59:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20190731:13:53:59:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Obtaining Segment details from master...
20190731:13:54:00:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Version: 'postgres (Greenplum Database) 5.20.0 build commit:459b837194a28ab2f32305a6b32080ce20100b7c'
20190731:13:54:00:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-There are 0 connections to the database
20190731:13:54:00:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Commencing Master instance shutdown with mode='immediate'
20190731:13:54:00:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Master host=dw-greenplum-1
20190731:13:54:00:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Commencing Master instance shutdown with mode=immediate
20190731:13:54:00:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Master segment instance directory=/data/gpdata/master/gpseg-1
20190731:13:54:01:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Attempting forceful termination of any leftover master process
20190731:13:54:01:017494 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Terminating processes for segment /data/gpdata/master/gpseg-1
20190731:13:54:01:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Starting gpstart with args: -a -l /home/gpadmin/gpAdminLogs -d /data/gpdata/master/gpseg-1
20190731:13:54:01:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Gathering information and validating the environment...
20190731:13:54:01:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Binary Version: 'postgres (Greenplum Database) 5.20.0 build commit:459b837194a28ab2f32305a6b32080ce20100b7c'
20190731:13:54:01:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Catalog Version: '301705051'
20190731:13:54:01:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Starting Master instance in admin mode
20190731:13:54:02:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20190731:13:54:02:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Obtaining Segment details from master...
20190731:13:54:02:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Setting new master era
20190731:13:54:02:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Master Started...
20190731:13:54:02:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Shutting down master
20190731:13:54:03:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Commencing parallel primary and mirror segment instance startup, please wait...
... 
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Process results...
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-   Successful segment starts                                            = 8
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-   Failed segment starts                                                = 0
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-   Skipped segment starts (segments are marked down in configuration)   = 0
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Successfully started 8 of 8 segment instances 
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:54:06:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Starting Master instance dw-greenplum-1 directory /data/gpdata/master/gpseg-1 
20190731:13:54:07:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Command pg_ctl reports Master dw-greenplum-1 instance active
20190731:13:54:07:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-No standby master configured.  skipping...
20190731:13:54:07:017520 gpstart:dw-greenplum-1:gpadmin-[INFO]:-Database successfully started
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Completed restart of Greenplum instance in production mode
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Scanning utility log file for any warning messages
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[WARN]:-*******************************************************
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[WARN]:-Scan of log file indicates that some warnings or errors
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[WARN]:-were generated during the array creation
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Please review contents of log file
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-/home/gpadmin/gpAdminLogs/gpinitsystem_20190731.log
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-To determine level of criticality
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-These messages could be from a previous run of the utility
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-that was called today!
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[WARN]:-*******************************************************
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Database instance successfully created
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-------------------------------------------------------
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-To complete the environment configuration, please 
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-update gpadmin .bashrc file with the following
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-1. Ensure that the greenplum_path.sh file is sourced
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-2. Add "export MASTER_DATA_DIRECTORY=/data/gpdata/master/gpseg-1"
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-   to access the Greenplum scripts for this instance:
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-   or, use -d /data/gpdata/master/gpseg-1 option for the Greenplum scripts
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-   Example gpstate -d /data/gpdata/master/gpseg-1
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Script log file = /home/gpadmin/gpAdminLogs/gpinitsystem_20190731.log
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-To remove instance, run gpdeletesystem utility
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-To initialize a Standby Master Segment for this Greenplum instance
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Review options for gpinitstandby
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-------------------------------------------------------
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-The Master /data/gpdata/master/gpseg-1/pg_hba.conf post gpinitsystem
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-has been configured to allow all hosts within this new
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-array to intercommunicate. Any hosts external to this
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-new array must be explicitly added to this file
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-Refer to the Greenplum Admin support guide which is
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-located in the /usr/local/greenplum-db/./docs directory
20190731:13:54:07:012489 gpinitsystem:dw-greenplum-1:gpadmin-[INFO]:-------------------------------------------------------

[gpadmin@dw-greenplum-1 gpconfigs]$ gpstate -s
20190731:13:59:02:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-Starting gpstate with args: -s
20190731:13:59:02:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-local Greenplum Version: 'postgres (Greenplum Database) 5.20.0 build commit:459b837194a28ab2f32305a6b32080ce20100b7c'
20190731:13:59:03:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-master Greenplum Version: 'PostgreSQL 8.3.23 (Greenplum Database 5.20.0 build commit:459b837194a28ab2f32305a6b32080ce20100b7c) on x86_64-pc-linux-gnu, compiled by GCC gcc (GCC) 6.2.0, 64-bit compiled on Jun 20 2019 04:46:42'
20190731:13:59:03:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-Obtaining Segment details from master...
20190731:13:59:03:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-Gathering data from segments...
. 
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:--Master Configuration & Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Master host                    = dw-greenplum-1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Master postgres process ID     = 17589
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Master data directory          = /data/gpdata/master/gpseg-1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Master port                    = 5432
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Master current role            = dispatch
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Greenplum initsystem version   = 5.20.0 build commit:459b837194a28ab2f32305a6b32080ce20100b7c
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Greenplum current version      = PostgreSQL 8.3.23 (Greenplum Database 5.20.0 build commit:459b837194a28ab2f32305a6b32080ce20100b7c) on x86_64-pc-linux-gnu, compiled by GCC gcc (GCC) 6.2.0, 64-bit compiled on Jun 20 2019 04:46:42
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Postgres version               = 8.3.23
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Master standby                 = No master standby configured
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-Segment Instance Status Report
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/primary/gpseg0
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 6000
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 4741
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Database status                   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-3
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/mirror/gpseg0
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 50000
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 24977
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Segment status                    = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/primary/gpseg1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 6001
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 4739
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Database status                   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-3
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/mirror/gpseg1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 50001
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 24979
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Segment status                    = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-3
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/primary/gpseg2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 6000
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 24978
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Database status                   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/mirror/gpseg2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 50000
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 4742
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Segment status                    = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-3
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/primary/gpseg3
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 6001
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Primary
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 24980
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Database status                   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Segment Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Hostname                          = dw-greenplum-2
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Address                           = sdw1
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Datadir                           = /data/gpdata/mirror/gpseg3
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Port                              = 50001
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Mirroring Info
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Current role                      = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Preferred role                    = Mirror
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Mirror status                     = Synchronized
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-   Status
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      PID                               = 4740
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Configuration reports status as   = Up
20190731:13:59:04:017908 gpstate:dw-greenplum-1:gpadmin-[INFO]:-      Segment status                    = Up
[gpadmin@dw-greenplum-1 gpconfigs]$ echo '
> host    all    all    0.0.0.0/0    md5' >> $MASTER_DATA_DIRECTORY/pg_hba.conf
[gpadmin@dw-greenplum-1 gpconfigs]$ gpstop -a -r
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Starting gpstop with args: -a -r
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Gathering information and validating the environment...
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Obtaining Greenplum Master catalog information
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Obtaining Segment details from master...
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Greenplum Version: 'postgres (Greenplum Database) 5.20.0 build commit:459b837194a28ab2f32305a6b32080ce20100b7c'
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-There are 0 connections to the database
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Commencing Master instance shutdown with mode='smart'
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Master host=dw-greenplum-1
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Commencing Master instance shutdown with mode=smart
20190731:14:01:34:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Master segment instance directory=/data/gpdata/master/gpseg-1
20190731:14:01:35:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Attempting forceful termination of any leftover master process
20190731:14:01:35:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Terminating processes for segment /data/gpdata/master/gpseg-1
20190731:14:01:35:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-No standby master host configured
20190731:14:01:35:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Targeting dbid [2, 6, 3, 7, 4, 8, 5, 9] for shutdown
20190731:14:01:35:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Commencing parallel primary segment instance shutdown, please wait...
20190731:14:01:35:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-0.00% of jobs completed
20190731:14:01:37:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-100.00% of jobs completed
20190731:14:01:37:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Commencing parallel mirror segment instance shutdown, please wait...
20190731:14:01:37:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-0.00% of jobs completed
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-100.00% of jobs completed
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-   Segments stopped successfully      = 8
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-   Segments with errors during stop   = 0
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-----------------------------------------------------
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Successfully shutdown 8 of 8 segment instances 
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Database successfully shutdown with no errors reported
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Cleaning up leftover gpmmon process
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-No leftover gpmmon process found
20190731:14:01:38:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Cleaning up leftover gpsmon processes
20190731:14:01:39:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-No leftover gpsmon processes on some hosts. not attempting forceful termination on these hosts
20190731:14:01:39:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Cleaning up leftover shared memory
20190731:14:01:39:018048 gpstop:dw-greenplum-1:gpadmin-[INFO]:-Restarting System...

```
