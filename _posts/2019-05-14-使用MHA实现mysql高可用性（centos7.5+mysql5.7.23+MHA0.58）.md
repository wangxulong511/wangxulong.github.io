使用MHA实现mysql高可用性（centos7.5+mysql5.7.23+MHA0.58）

> https://blog.csdn.net/dayi_123/article/details/83690608  


# mysql数据库高可用架构-MHA-0.56/0.57详解
> https://blog.51cto.com/arthur376/1812640  

一、MHA概述
1、MHA

         MHA（Master High Availability）事由日本人DeNA开发的一套MySQL高可用性环境下故障切换和主从提升的软件，目前在MySQL高可用方面是一个相对成熟的解决方案。在MySQL故障切换过程中，MHA能做到在0~30秒之内自动完成数据库的故障切换操作，并且在进行故障切换的过程中，MHA能在最大程度上保证数据的一致性，以达到真正意义上的高可用。

2、MHA的主要功能

（1）自动故障检测和自动故障转移

（2）交互式（手动）故障转移

（3）在线切换Master到不同的主机

3、MHA的优势

（1）自动故障转移快

（2）主库崩溃不存在数据一致性问题

（3）配置不需要对当前mysql环境做重大修改

（4）不需要添加额外的服务器(仅一台manager就可管理上百个replication)

（5）性能优秀，可工作在半同步复制和异步复制

（6）只要replication支持的存储引擎，MHA都支持，不会局限于innodb

4、MHA的组成部分

         MHA由Manager节点和Node节点组成；MHA Manager可以单独部署在一台独立的机器上管理多个master-slave集群，也可以部署在一台slave节点上。MHA Node运行在每台MySQL服务器上。

5、MHA工作原理

（1）MHA Manager会定时探测集群中的master节点

（2）当master出现故障时，从宕机崩溃的master保存二进制日志事件（binlog events）;

（3）识别含有最新更新的slave；

（4）应用差异的中继日志（relay log）到其他的slave；

（5）应用从master保存的二进制日志事件（binlog events）；

（6）提升一个slave为新的master，使其他的slave连接新的master进行复制；

二、MHA安装部署
1、环境准备

（1）环境准备
```
10.192.130.200 130-centos200
10.192.130.69 130-test-redis69
10.192.130.68 130-test-redis68
```

3、安装MHA

（1）在各节点上安装mha4mysql-node

         在四台机器上上分别安装mha4mysql-node。

         mha4mysql-node下载地址：https://github.com/yoshinorim/mha4mysql-node/releases/tag/v0.58

         mha4mysql-manager下载地址：https://github.com/yoshinorim/mha4mysql-manager/releases/tag/v0.58

（3）在管理节点上安装manage节点

# 在管理节点安装依赖软件
]# yum install -y perl-Config-Tiny perl-Log-Dispatch  perl-Parallel-ForkManager  perl-Time-HiRes
# 在管理节点安装mha4mysql-manager
]# rpm -ivh mha4mysql-manager-0.58-0.el7.centos.noarch.rpm
