---
layout:     post
title:      MySQL + keepalive高可用搭建
subtitle:   MySQL + keepalive高可用搭建(如果不踩坑的话🙈🙊🙉)
date:       2017-02-06
author:     BY
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - MySQL+keepalive
---

# MySQL + keepalive高可用搭建

> https://www.cnblogs.com/ywrj/p/9483427.html

系统环境：centos6.8  

Ip：
10.10.130.111 主库  
10.10.130.112 从库  

VIP(虚拟ip/浮动ip)10.10.130.110  

软件版本：mysql 5.7.22  
Keepalived 1.2.13   
## 安装MySQL 主从

## 安装keepalived
yum install keepalived

## 配置keeplived

10.10.130.111 主库
修改keepalived的配置文件
```
# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1 
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 98
    advert_int 5
    nopreempt 
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        #192.168.200.16
        #192.168.200.17
        #192.168.200.18
        10.10.130.110
    }
}

virtual_server 10.10.130.110 3306 {
    delay_loop 6 
    lb_algo wrr 
    lb_kind DR 
    persistence_timeout 60 
    protocol TCP 
    
    real_server 10.10.130.111 3306 { 
        weight 98 
        notify_down /data/sh/mysql.sh 
        TCP_CHECK { 
            connect_timeout 10 
	    nb_get_retry 3 
            delay_before_retry 3 
            connect_port 3306 
	} 
    }

}

保存退出

```

10.10.130.112 从库
修改keepalived的配置文件

```
# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1 
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 98
    advert_int 5
    nopreempt 
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        #192.168.200.16
        #192.168.200.17
        #192.168.200.18
        10.10.130.110
    }
}

virtual_server 10.10.130.110 3306 {
    delay_loop 6 
    lb_algo wrr 
    lb_kind DR 
    persistence_timeout 60 
    protocol TCP 
    
    real_server 10.10.130.112 3306 { 
        weight 98 
        notify_down /data/sh/check_mysql.sh 
        TCP_CHECK { 
            connect_timeout 10 
	    nb_get_retry 3 
            delay_before_retry 3 
            connect_port 3306 
	} 
    }

}
   
```

保存退出

上面的配置简单说明：
state BACKUP  表示为备模式，nopreempt 为不抢占，双方都配为备模式，并且不抢占，可以避免“脑裂”问题，priority 100 为权重，数字越高，权重越高，当双方都配为BACKUP模式，并且配置nopreempt时，keepalived靠这个数字来判断谁是主，谁是备  
主从都要创建/data/sh/mysql.sh脚本  
mkdir /data/sh  
vim /data/sh/check_mysql.sh 加入以下内容：  
```
#！/bin/bash
/etc/init.d/keepalived  stop  
```

或者 

```
#!/bin/bash  
MYSQL=/usr/local/mysql/bin/mysql  
MYSQL_HOST=localhost 
MYSQL_USER=root 
MYSQL_PASSWORD=root
CHECK_TIME=3  
#mysql  is working MYSQL_OK is 1 , mysql down MYSQL_OK is 0  
MYSQL_OK=1 
function check_mysql_health (){  
$MYSQL -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD -e "show status;" >/dev/null 2>&1  
if [ $? = 0 ] ;then  
     MYSQL_OK=1 
else  
     MYSQL_OK=0 
fi  
     return $MYSQL_OK  
}  
while [ $CHECK_TIME -ne 0 ]  
do  
     let "CHECK_TIME-=1" 
     check_mysql_health  
     if [ $MYSQL_OK = 1 ] ; then  
          CHECK_TIME=0 
          exit 0  
     fi  
 
     if [ $MYSQL_OK -eq 0 ] &&  [ $CHECK_TIME -eq 0 ]  
     then  
          /etc/init.d/keepalived stop  
     exit 1   
     fi  
     sleep 1  
done 
```
chmod  755  /data/sh/check_mysql.sh  
modprobe  ip_vs  #加载ip_vs模块 虚拟IP要用  
lsmod |  grep  ip_vs  查看ip_vs模块有没有加载，如果看到下面的内容，就表示加载成功：  


/etc/init.d/mysqld  start  #启动mysql
/etc/init.d/keepalive  start   #启动keepalived
在36上的操作和38上一样，只是keepalived.conf配置文件里priority 100 改为 priority 98
 
real_server 10.10.130.111 改为 10.10.130.112其他不变。
 
查看VIP情况命令: ip a

 
至此mysql的主从高可用就做好了，可以在111上测试挺掉mysql服务，看看vip会不会漂移到112上，一般来说都是没问题的，如果有问题，请检查你的配置，步骤是不是有错误，还有selinux，防火墙是否关闭等



2019-8-21 
听群里的大佬说只做VIP飘移不需要配置 virtual_server，LVS 做负载时使用，大佬原话(vip只做主备切换不配合lvs的话，我都是这样配置的，不用写virtual server)

 
```
# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {

    router_id MySQL_HA
	vrrp_skip_check_adv_addr
	#vrrp_strict
	#vrrp_garp_interval  0
	#vrrp_gna_interval  0

}

varrp_script check_script  {
    script "/data/sh/mysql.sh"
    #script "/etc/keepalived/scripts/check_mysql.sh"
    interval 3   
}

vrrp_instance VI_1 {

   state BACKUP    

   interface eth0

   virtual_router_id 245

   priority  100

   advert_int 2
   
   nopreempt
   
   track_script {
   
   check_script
   
   }
   
   authentication {

        auth_type PASS

        auth_pass 1111   

   }

   virtual_ipaddress {

        10.10.130.110/24 dev eth0 scope global label eth0:1

        }

   }
```

