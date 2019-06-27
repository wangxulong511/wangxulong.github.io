# Clickhouse安装及使用

```
一、简介
Yandex在2016年6月15日开源了一个数据分析的数据库，名字叫做ClickHouse，这对保守俄罗斯人来说是个特大事。更让人惊讶的是，这个列式存储数据库的跑分要超过很多流行的商业MPP数据库软件，例如Vertica。如果你没有听过Vertica，那你一定听过 Michael Stonebraker，2014年图灵奖的获得者，PostgreSQL和Ingres发明者（Sybase和SQL Server都是继承Ingres而来的）, Paradigm4和SciDB的创办者。Michael Stonebraker于2005年创办Vertica公司，后来该公司被HP收购，HP Vertica成为MPP列式存储商业数据库的高性能代表，Facebook就购买了Vertica数据用于用户行为分析。简单的说，ClickHouse作为分析型数据库，有三大特点：一是跑分快，二是功能多，三是文艺范 
官网地址：https://clickhouse.yandex/ 
官方文档：https://clickhouse.yandex/docs/en/single/ 
 


二、安装配置
2.1 支持的操作系统和硬件环境
只要是Linux，64位都可以。 
优先支持Ubuntu，Ubuntu有官方编译好的安装包可以使用。 
其次是CentOS和RedHat，有第三方组织altinity编译好的rpm包可以使用。 
如果是其他Linux系统，需要自己编译源码。 
而且，机器的CPU必须支持SSE 4.2指令集。 
[root@localhost ~]# grep -q sse4_2 /proc/cpuinfo && echo “SSE 4.2 supported” || echo “SSE 4.2 not supported” 
SSE 4.2 supported 
 

2.2 单机版
2.2.1 联网安装
2.2.1.1 按照官网上的步骤安装(安装环境：64位的Ubuntu 14.04 LTS）：
命令1：更新keyserver 
sudo apt-key adv –keyserver keyserver.ubuntu.com –recv E0C56BD4

命令2：增加仓库地址 
如果是Ubuntu 16.04 Xenial，执行这句

sudo apt-add-repository "deb http://repo.yandex.ru/clickhouse/xenial stable main"
1
如果是Ubuntu 14.04 Trusty，执行这句

sudo apt-add-repository "deb http://repo.yandex.ru/clickhouse/trusty stable main"
1
如果是Ubuntu 12.04 Precise，执行这句

sudo apt-add-repository "deb http://repo.yandex.ru/clickhouse/precise stable main"
1
命令3：更新软件列表 
sudo apt-get update

命令4：安装server和client 
sudo apt-get install clickhouse-server-common clickhouse-client -y

命令5：修改数据目录 
参考运维指南->参数设置->设置数据目录

命令6：启动sever服务 
sudo service clickhouse-server start 
 

2.2.1.2 Debian-8.8.0-amd64：
root@debian:~# apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 E0C56BD4
root@debian:~# vi /etc/apt/sources.list
添加：deb http://repo.yandex.ru/clickhouse/trusty stable main
root@debian:~# apt-get update
root@debian:~# apt-get install clickhouse-server-common clickhouse-client -y
root@debian:~# service clickhouse-server start
1
2
3
4
5
6
注意：可能是因为网络的原因，我在家中执行第一步就成功，在公司却失败了，报下面这个错

gpg: requesting key E0C56BD4 from hkp server keyserver.ubuntu.com
?: keyserver.ubuntu.com: Connection refused
gpgkeys: HTTP fetch error 7: couldn't connect: Connection refused
gpg: no valid OpenPGP data found.
gpg: Total number processed: 0
1
2
3
4
5
解决：执行以下命令 
sudo apt-key adv –recv-keys –keyserver hkp://keyserver.ubuntu.com:80 E0C56BD4 
 

2.2.2 离线安装
2.2.2.1 Ubuntu
(1)下载安装介质 
安装ClickHouse需要用到3个安装包： 
clickhouse-client 
clickhouse-server-base 
clickhouse-server-common 
下载地址：http://repo.yandex.ru/clickhouse/deb/stable/main/

(2)把下载好的三个deb安装包上传到服务器的/tmp目录 
clickhouse-client_1.1.54362_amd64.deb 
clickhouse-server-base_1.1.54362_amd64.deb 
clickhouse-server-common_1.1.54362_amd64.deb

(3)执行安装 
sudo dpkg -i clickhouse-server-base_1.1.54362_amd64.deb 
sudo dpkg -i clickhouse-server-common_1.1.54362_amd64.deb 
sudo dpkg -i clickhouse-client_1.1.54362_amd64.deb

(4)启动clickhouse-sever服务 
sudo service clickhouse-server start 
 

2.2.2.2 CentOS 6.5（RedHat 6.6也适用）安装方法（Centos 7.2类似）
注意：按照官网https://clickhouse.yandex/docs/en/getting_started/的方法居然成功不了，好像是没更新，repo文件中的网址都找不到 
 
(1)下载介质 
安装介质可以在这里找到：https://packagecloud.io/altinity/clickhouse（下载最新的54362，因为我按网上的一些旧文章下载54236总是报错） 

注：我当时是下载了http://repo.red-soft.biz/repos/clickhouse/stable/中的rpm老包，结果总是报这个错，我用yum安装了很多包可依然是报这个错，最终我只能放弃了（后来换为Centos6.8就行，原因就是Centos6.8默认在/usr/lib64目录下就是libbfd-2.20.51.0.2-5.44.el6.so，而Centos6.5的/usr/lib64目录下默认是libbfd-2.20.51.0.2-5.36.el6.so，Redhat6.6默认是libbfd-2.20.51.0.2-5.42.el6.so。即使是yum install -y binutils后却更新为libbfd-2.20.51.0.2-5.47.el6_9.1.so；或者将Centos6.8下的libbfd-2.20.51.0.2-5.44.el6.so传到Centos6.5或Redhat6.6的相应目录下并把原来旧的删除掉还是不行） 
[root@localhost clickhouse]# rpm -ivh clickhouse-server-1.1.54236-4.el6.x86_64.rpm

error: Failed dependencies:
        libbfd-2.20.51.0.2-5.44.el6.so()(64bit) is needed by clickhouse-server-1.1.54236-4.el6.x86_64
1
2
(2)设置系统参数 
CentOS取消打开文件数限制 
在/etc/security/limits.conf、/etc/security/limits.d/90-nproc.conf这2个文件的末尾加入一下内容： 
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072

CentOS取消SELINUX 
修改/etc/selinux/config中的SELINUX=disabled后重启

关闭防火墙 
Centos 6.X： 
service iptables stop 
service ip6tables stop 
Centos 7.X： 
chkconfig iptables off 
chkconfig ip6tables off

(3)安装 
Centos 6.5： 
rpm -ivh clickhouse-server-common-1.1.54362-1.el6.x86_64.rpm 
[root@localhost clickhouse]# rpm -ivh clickhouse-server-1.1.54362-1.el6.x86_64.rpm

error: Failed dependencies:
        libodbc.so.2()(64bit) is needed by clickhouse-server-1.1.54362-1.el6.x86_64
1
2
解决：yum install -y unixODBC 
rpm -ivh clickhouse-debuginfo-1.1.54362-1.el6.x86_64.rpm 
rpm -ivh clickhouse-client-1.1.54362-1.el6.x86_64.rpm 
rpm -ivh clickhouse-test-1.1.54362-1.el6.x86_64.rpm（这个rpm包不下载安装也可）

Centos 7.2： 
rpm -ivh clickhouse-server-common-1.1.54236-4.el7.x86_64.rpm 
[root@localhost clickhouse]# rpm -ivh clickhouse-server-1.1.54236-4.el7.x86_64.rpm

error: Failed dependencies:
        libicudata.so.50()(64bit) is needed by clickhouse-server-1.1.54236-4.el7.x86_64
        libicui18n.so.50()(64bit) is needed by clickhouse-server-1.1.54236-4.el7.x86_64
        libicuuc.so.50()(64bit) is needed by clickhouse-server-1.1.54236-4.el7.x86_64
        libltdl.so.7()(64bit) is needed by clickhouse-server-1.1.54236-4.el7.x86_64
        libodbc.so.2()(64bit) is needed by clickhouse-server-1.1.54236-4.el7.x86_64
1
2
3
4
5
6
解决：yum install -y unixODBC libicudata 
rpm -ivh clickhouse-debuginfo-1.1.54236-4.el7.x86_64.rpm 
rpm -ivh clickhouse-compressor-1.1.54236-4.el7.x86_64.rpm 
rpm -ivh clickhouse-client-1.1.54236-4.el7.x86_64.rpm

注意：在Centos 7.2中安装54362的时候多了个这个依赖缺失 
[root@localhost clickhouse]# rpm -ivh clickhouse-server-1.1.54362-1.el7.x86_64.rpm

error: Failed dependencies:
        libcrypto.so.10(OPENSSL_1.0.2)(64bit) is needed by clickhouse-server-1.1.54362-1.el7.x86_64
1
2
解决：yum install -y openssl 
号外：一开始我用yum安装好openssl后又报这个错咋也解决不了。后来发现我下载的包有问题才4M多，正常应该是13M多，这个外国网址特难下载，下的慢还给我下了一半居然也每个提示真是醉了。 
[root@localhost clickhouse]# rpm -ivh clickhouse-server-1.1.54362-1.el7.x86_64.rpm

Preparing...                          ################################# [100%]
Updating / installing...
   1:clickhouse-server-1.1.54362-1.el7################################# [100%]
error: unpacking of archive failed on file /usr/bin/clickhouse;5988c8ac: cpio: read failed - No such file or directory
error: clickhouse-server-1.1.54362-1.el7.x86_64: install failed
1
2
3
4
5
rpm -ivh clickhouse-client-1.1.54362-1.el7.x86_64.rpm（剩下的clickhouse-test-1.1.54362-1.el7.x86_64.rpm和clickhouse-debuginfo-1.1.54362-1.el7.x86_64.rpm可以不安装）

(4)启动连接 
service clickhouse-server start 
clickhouse-client 
 

2.2.3 Docker安装方法
参考网址：https://hub.docker.com/r/yandex/clickhouse-server/ 
启动服务端： 
sudo docker run -d –name some-clickhouse-server –ulimit nofile=262144:262144 yandex/clickhouse-server 
从本地客户端连接到它： 
sudo docker run -it –rm –link some-clickhouse-server:clickhouse-server yandex/clickhouse-client –host clickhouse-server（这行运行完就直接进入到了clickhouse的交互式客户端了）

容器公开了用于HTTP接口的8123端口和用于本地客户端的9000端口。用自定义配置启动服务端实例：
sudo docker run -d --name some-clickhouse-server --ulimit nofile=262144:262144 -v /path/to/your/config.xml:/etc/clickhouse-server/config.xml yandex/clickhouse-server
1
2
2.3 集群版（后续完善）
 

2.4 进入交互式客户端
hui@root-virtual-machine:~$ clickhouse-client

ClickHouse client version 1.1.54343.
Connecting to localhost:9000.
Connected to ClickHouse server version 1.1.54343.

:) show databases;

SHOW DATABASES

┌─name────┐
│ default │
│ system  │
└─────────┘

2 rows in set. Elapsed: 0.059 sec.

:) select now();

SELECT now()

┌───────────────now()─┐
│ 2018-03-14 11:23:44 │
└─────────────────────┘

1 rows in set. Elapsed: 0.183 sec. 

启动失败可以查看日志，日志的目录默认为/var/log/clickhouse-server/下。

hui@root-virtual-machine:~$ ls /var/log/clickhouse-server/
clickhouse-server.err.log  clickhouse-server.log  stderr  stdout
1
2
 

三、运维指南
3.1 服务起停
停止： 
service clickhouse-server stop

启动： 
service clickhouse-server start

用clickhouse-client连接本机clickhouse-server服务器： 
Clickhouse-client

用本机clickhouse-client连接远程clickhouse-server服务器： 
clickhouse-client –host 192.168.3.54 –port 9000 –database default–user default –password “” 
 

3.2 参数配置
3.2.1 放开远程访问
vi /etc/clickhouse-server/config.xml 
修改服务器的配置文件/etc/clickhouse-server/config.xml，第65行，放开注释即可，修改之后的内容如下： 


3.2.2 内存限制设置
vi /etc/clickhouse-server/users.xml 


3.2.3 设置数据目录
vi /etc/clickhouse-server/config.xml 
 
 

3.3 常见FAQ
Q1：无法在本地连接ClickHouse，报错：Connecton reject。 
A1：clickhouse默认不放开远程访问，可以通过修改配置文件的方式放开。 
修改服务器的配置文件/etc/clickhouse-server/config.xml，第65行，放开注释即可。 
修改之后，重启服务： 
service clickhouse-server stop 
service clickhouse-server start

Q2：修改metrika.xml里面的shard或者replica参数之后，数据库无法启动； 
A2：如果之前已经建了Replica表，那么必须在修改上述两个参数之前把已经创建好的Replica表删掉，才能去修改，否则会无法启动。报错日志放在/var/log/clickhouse-server/下。 
 

四、开发技巧
4.1 登陆
用Dbeaver可以直接连接clickhouse，类似这样： 
 
如果是本地连接服务器，必须先放开远程访问的权限。具体看FAQ的“无法在本地连接clickhouse”。 
注：Dbeaver在第一次连接clickhouse的时候在能联网的状态下会自动加载clickhouse-jdbc的jar包，我觉的这点非常的好。加载后的存放目录为C:\Users\hui.dbeaver-drivers\maven\maven-central-unsecure\ru.yandex.clickhouse\clickhouse-jdbc-0.1.38.jar 
如果你想自己手动下载的话需要用maven编译，下载地址为：https://github.com/yandex/clickhouse-jdbc 
 

4.2 建表
CREATE TABLE code_province( \
    state_province        String, \
    province_name         String, \
    create_date           date \
) ENGINE = MergeTree(create_date, (state_province), 8192);
1
2
3
4
5
ENGINE：是表的引擎类型，最常用的MergeTree。还有一个Log引擎也是比较常用。MergeTree要求有一个日期字段，还有主键。Log没有这个限制。 
create_date：是表的日期字段，一个表必须要有一个日期字段。 
State_province：是表的主键，主键可以有多个字段，每个字段用逗号分隔 
8192：是索引粒度，用默认值8192即可。 
 

4.3 导数
4.3.1 普通的CSV文件导入
cat > code_province.csv << EOF
WA,WA_NAME,2017-12-25
CA,CA_NAME,2017-12-25
OR,OR_NAME,2017-12-25
EOF
1
2
3
4
5
–导数 
clickhouse-client –query “INSERT INTO default.code_province FORMAT CSV” < code_province.csv 
–或者用管道的方式 
cat code_province.csv | clickhouse-client –query “INSERT INTO default.code_province FORMAT CSV” 
 

4.3.2 特殊的CSV导入（包含回车换行、转义符等）
–建表（sql语句） 
CREATE TABLE default.test3 ( id1 UInt32, id2 Float32, name1 String, name2 String, date1 Date, date2 DateTime) ENGINE = Log;

–测试数据（Linux命令） 
root@smartbi-virtual-machine:/data/test# cat test3.csv 
1,123.456,”abc123”,”abc”“’ 
123”,2017-05-06,2017-05-06 07:08:09

–执行导入（Linux命令） 
clickhouse-client –query “INSERT INTO default.test3 FORMAT CSV” < test3.csv

–查看结果（sql语句）

:) select * from test3;

┌─id1─┬─────id2─┬─name1──┬─name2───────┬──────date1─┬───────────────date2─┐
│   1 │ 123.456 │ abc123 │ abc"\'\n123 │ 2017-05-06 │ 2017-05-06 07:08:09 │
└─────┴─────────┴────────┴─────────────┴────────────┴─────────────────────┘
```

来源：CSDN 
> 原文：https://blog.csdn.net/m0_37739193/article/details/79611560 


使用ClickHouse一键接管MySQL数据分析
> 原文：https://www.jianshu.com/p/5bfb043a075d
