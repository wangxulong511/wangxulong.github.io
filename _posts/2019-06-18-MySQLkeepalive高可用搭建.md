# MySQL + keepalive高可用搭建

系统环境：centos6.8  

Ip：
10.101.130.111 主库  
10.101.130.112 从库  

VIP(虚拟ip/浮动ip)10.101.130.110  

软件版本：mysql 5.7.22  
Keepalived 1.2.13   
## 安装MySQL 主从

## 安装keepalived
yum install keepalived

## 配置keeplived

10.101.130.111 主库
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
        10.101.130.110
    }
}

virtual_server 10.101.130.110 3306 {
    delay_loop 6 
    lb_algo wrr 
    lb_kind DR 
    persistence_timeout 60 
    protocol TCP 
    
    real_server 10.101.130.111 3306 { 
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

10.101.130.112 从库
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
        10.101.130.110
    }
}

virtual_server 10.101.130.110 3306 {
    delay_loop 6 
    lb_algo wrr 
    lb_kind DR 
    persistence_timeout 60 
    protocol TCP 
    
    real_server 10.101.130.112 3306 { 
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
   
```

保存退出

上面的配置简单说明：
state BACKUP  表示为备模式，nopreempt 为不抢占，双方都配为备模式，并且不抢占，可以避免“脑裂”问题，priority 100 为权重，数字越高，权重越高，当双方都配为BACKUP模式，并且配置nopreempt时，keepalived靠这个数字来判断谁是主，谁是备
主从都要创建/data/sh/mysql.sh脚本
mkdir /data/sh
vim /data/sh/mysql.sh 加入以下内容：
#！/bin/bash
/etc/init.d/keepalived  stop
 
chmod  755  /data/sh/mysql.sh
modprobe  ip_vs  #加载ip_vs模块 虚拟IP要用
lsmod |  grep  ip_vs  查看ip_vs模块有没有加载，如果看到下面的内容，就表示加载成功：


/etc/init.d/mysqld  start  #启动mysql
/etc/init.d/keepalive  start   #启动keepalived
在36上的操作和38上一样，只是keepalived.conf配置文件里priority 100 改为 priority 98
 
real_server 10.101.130.111 改为 10.101.130.112其他不变。
 
查看VIP情况命令: ip a

 
至此mysql的主从高可用就做好了，可以在111上测试挺掉mysql服务，看看vip会不会漂移到112上，一般来说都是没问题的，如果有问题，请检查你的配置，步骤是不是有错误，还有selinux，防火墙是否关闭等
