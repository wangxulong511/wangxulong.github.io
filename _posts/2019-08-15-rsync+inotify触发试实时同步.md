# rsync+inotify触发试实时同步

一、rsync的优点与不足

 与传统的cp、tar备份方式相比，rsync具有安全性高、备份迅速、支持增量备份等优点，通过rsync可以解决对实时性要求不高的数据备份需求，例如定期的备份文件服务器数据到远端服务器，对本地磁盘定期做数据镜像等。

 随着应用系统规模的不断扩大，对数据的安全性和可靠性也提出的更好的要求，rsync在高端业务系统中也逐渐暴露出了很多不足，首先，rsync同步数据时，需要扫描所有文件后进行比对，进行差量传输。如果文件数量达到了百万甚至千万量级，扫描所有文件将是非常耗时的。而且正在发生变化的往往是其中很少的一部分，这是非常低效的方式。其次，rsync不能实时的去监测、同步数据，虽然它可以通过linux守护进程的方式进行触发同步，但是两次触发动作一定会有时间差，这样就导致了服务端和客户端数据可能出现不一致，无法在应用故障时完全的恢复数据。基于以上原因，rsync+inotify组合出现了！

 

二、 初识inotify

 Inotify 是一种强大的、细粒度的、异步的文件系统事件监控机制，linux内核从2.6.13起，加入了Inotify支持，通过Inotify可以监控文件系统中添加、删除，修改、移动等各种细微事件，利用这个内核接口，第三方软件就可以监控文件系统下文件的各种变化情况，而inotify-tools就是这样的一个第三方软件。

在上面章节中，我们讲到，rsync可以实现触发式的文件同步，但是通过crontab守护进程方式进行触发，同步的数据和实际数据会有差异，而inotify可以监控文件系统的各种变化，当文件有任何变动时，就触发rsync同步，这样刚好解决了同步数据的实时性问题。

 

操作系统centos7.4

\#####inotify+rsync########

 

服务端：10.101.130.193  

客户端：10.101.130.191

 

发布服务器10.101.130.193  

将目录  /data/backup/

实时同步到192.168.232.134

目录 /data/backup/





客户端：10.101.130.191



1.客户端安装配置 rsync

[root@www-1~]# yum install rsync

修改 rsyncd.conf配置

\#vi /etc/rsyncd.conf

```
uid = root

gid = root

use chroot = no

max connections = 10

timeout = 600

strict modes = yes

pid file = /var/run/rsyncd.pid

lock file = /var/run/rsync.lock

log file = /var/log/rsyncd.log

 

[mysqldata]

path = /data/backup

ignore errors

read only = false

list = false

hosts allow = 10.101.130.0/255.255.255.0

hosts deny = 0.0.0.0/32

auth users = rsync_backup

secrets file = /etc/rsyncd.passwd
```



\#echo "rsync_backup:654321" >/etc/rsync.passwd   #//写入密码到密码文件

\#chmod 600 /etc/rsync.passwd            #//设置密码文件权限为600读写

\#cat /etc/rsync.passwd      #//查看密码是否成功写入密码文件

\#ll /etc/rsync.passwd       #//查看密码文件权限是否为600

\#/usr/bin/rsync --daemon          //启动rsync

\#echo "/usr/bin/rsync --daemon" >>/etc/rc.local  //将文件写入自启动文件中

\#cat /etc/rc.local|grep daemon //查看是否成功写入

\#ps -ef|grep rsync  //显示现行终端机下的所有程序并过滤出与rsync有关的

\#pkill rsync    //结束rsync

\#/usr/bin/rsync --daemon    //启动rsync







[10.101.130.193]服务端

1、yum 安装rsync

[root@www-2 ~]# yum install gcc

[root@www-2 ~]# yum install rsync

2、配置rsync.passwd

\#echo "654321" >/etc/rsync.passwd    //将密码654321写入密码文件中

\#chmod 600 /etc/rsync.passwd   //设置密码文件权限为600

\#cat /etc/rsync.passwd        //查看密码是否成功写入密码文件

\#ll /etc/rsync.passwd       //查看密码文件权限是否成功写入600权限



3、安装inotify-tools

inotify-tools-3.14 百度网盘源码包下载

链接：https://pan.baidu.com/s/198gxtaxJuCzz96EWGlErFw 
提取码：x1sm 

可以到http://inotify-tools.sourceforge.net/下载相应的inotify-tools版本，然后开始编译安装：	

 

\#tar zxvf inotify-tools-3.14.tar.gz

\#cd inotify-tools-3.14

#./configure --prefix=/usr/local/inotify-tools

#make && make install

[root@www-2  inotify-tools-3.14]# ll /usr/local/inotify-tools/bin/inotifywa*
-rwxr-xr-x. 1 root root 60896 Aug 15 10:30 /usr/local/inotify-tools/bin/inotifywait
-rwxr-xr-x. 1 root root 55176 Aug 15 10:30 /usr/local/inotify-tools/bin/inotifywatch



4、配置内容发布节点-----服务端193

inotify_rsync.sh

rsync同步是否有效,同步目录

```
#/usr/bin/rsync -zrtopg --delete --password-file=/etc/rsyncd.passwd  /data/backup/  rsync_backup@10.101.130.191::mysqldata
```



5、配置inotify-tools脚本

#cd 

#vim inotify_rsync.sh

```
#!/bin/bash
host=10.101.130.191 #server的ip(备份服务器)
src=/data/backup/
des=mysqldata
user=rsync_backup

/usr/local/inotify-tools/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e modify,delete,create,attrib $src | while read files  
do
  /usr/bin/rsync -zrtopg --delete --password-file=/etc/rsyncd.passwd $src $user@$host::$des
  echo "${files} was rsynced" >>/root/rsync.log 2>&1
done


```

7、启动

\#nohup /root/inotify_rsync.sh &



8、测试是文件是否会同步到客户端

\# cd /data/backup/

\# touch test.txt



Rsync ERROR: auth failed on module解决方法，password-file文件权限导致

```
https://www.jb51.net/article/41417.htm
```





