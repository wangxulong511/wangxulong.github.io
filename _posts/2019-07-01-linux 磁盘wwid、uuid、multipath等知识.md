# linux 磁盘wwid、uuid、multipath等知识

linux 磁盘wwid、uuid、multipath等知识
2018年07月12日 19:15:59 tuolaji8 阅读数：1567
1、wwid:

根据SCSI标准，每个SCSI磁盘都有一个WWID。类似于网卡的MAC地址，要求是独一无二。通过WWID标示SCSI磁盘就可以保证磁盘路径永久不变，Linux系统上/dev/disk/by-id目录包含每个SCSI磁盘WWID访问路径。

提示：Linux自带的device-mapper-multipath工具就是通过WWID来探测SCSI磁盘路径，可以将同一设备多条路径合并，并在/dev/mapper/下面创建新的设备路径。通过multipath –l可以看到WWID与 磁盘路径。

scsi_id命令执行后，只有磁盘、存储盘才可以显示wwid，多路径的存储盘显示的wwid一样
本地磁盘分区、存储盘分区都没有wwid
存储盘分区后，存储盘本身的wwid不变
存储盘分区且格式化后，存储盘本身的wwid不变

[root@mesdg ~]# /lib/udev/scsi_id -g -u /dev/sdi
3600a098038303867635d4a48624e5465

2、uuid:

UUID是有文件系统在创建时候生成的，用来标记文件系统，类似WWID一样也是独一无二的。因此使用UUID来标示SCSI磁盘，也能保证路径是永久不变的。Linux上/dev/disk/by-uuid可以看到每个已经创建文件系统的磁盘设备以及与/dev/sd之间的映射关键。
注意：Linux自带的md和LVM工具也会在SCSI磁盘上面写入UUID信息。

blkid命令执行后，磁盘、本地磁盘分区、存储盘都可以显示uuid，且uuid之间有-横杠，多路径的存储盘显示的uuid一样
存储盘进行分区但是不格式的话，分区没有uuid，存储盘本身的uuid不变
存储盘进行分区且格式的话，分区有uuid，存储盘本身的uuid改变了

[root@mesdg ~]# blkid|grep /dev/sdi
/dev/sdi: UUID="d987b768-cbd0-4a9a-a40d-58ec701853a9" TYPE="ext4"

得出结论wwid更稳定，一个存储盘格式化后wwid都不会改变，存储盘本身的uuid不变，但是格式化后会发现存储盘分区有了uuid，也更了解了，在应用服务器上/ets/fstab里面只能使用uuid，不能使用wwid，因为分区格式化后才有uuid


在应用服务器上/ets/fstab里面只能使用uuid，不能使用wwid

3、lun:

一个lun对应一个uuid和wwid，lun就是在存储服务器上划出来的一个分区，给相应的应用服务器当存储盘使用，UUID在存储服务器上和应用服务器上显示不一样（因为存储服务器和应用服务器的OS不一样）。

在存储服务器端uuid显示为fed94edc-896e-42cd-a3fe-8b2499097b88
ClusterB::> lun show -vserver wwxt02 -path * -fields uuid
vserver path                  uuid                                 
------- --------------------- ------------------------------------ 

wwxt02  /vol/wwxt02_data/mesdg fed94edc-896e-42cd-a3fe-8b2499097b88

在应用服务器端uuid显示为d987b768-cbd0-4a9a-a40d-58ec701853a9
[root@mesdg ~]# sudo blkid |grep /dev/mapper

/dev/mapper/3600a098038303867635d4a48624e5465: UUID="d987b768-cbd0-4a9a-a40d-58ec701853a9" TYPE="ext4"



同一lun挂载两台不同服务器上，两台服务器的操作系统一样，显示的wwid一样,如下fdisk -l |grep /dev/mapper

[root@mesdg fc_host]# fdisk -l |grep /dev/mapper
Disk /dev/mapper/3600a098038303867635d4a48624e5464: 6597.1 GB, 6597069766656 bytes

[root@bidg host11]# fdisk -l |grep /dev/mapper
Disk /dev/mapper/3600a098038303867635d4a48624e5464: 6597.1 GB, 6597069766656 bytes
4、multipath
在应用服务器端，同一个lun或wwid或uuid对应多个路径，优点是当某个hba卡坏掉的情况下，应用服务器仍可以访问存储服务器

一个lun配置了多少路径的算法=lun上绑定的某一台应用服务器hba数*存储机头数*光纤交换机数

只有1个交换机的情况下
一套存储服务器的机头柜里面有2台机头，应用服务器2个hba卡
在存储里面划分lun时对应2个hba卡，则应用服务器可以看到这个lun对应4个路径
在存储里面划分lun时对应1个hba卡，则应用服务器可以看到这个lun对应2个路径

2个交换机的情况下
一套存储服务器的机头柜里面有2台机头，应用服务器2个hba卡
在存储里面划分lun时对应2个hba卡，则应用服务器可以看到这个lun对应8个路径

在存储里面划分lun时对应1个hba卡，则应用服务器可以看到这个lun对应4个路径

一个lun可以被多少应用服务器访问的设置
主要看存储服务器上lun配置的hba信息，绑定了多少应用服务器的hba卡，就可以被多少应用服务器访问
不过尽量不要这样，一个lun就给一个应用服务器使用好了，多个应用服务器的话同时使用同一个个lun的话，容易会损坏数据。 
因为各个应用服务器之间又没有什么关联的，A应用服务器服务器删除了B应用服务器存放在lun上的数据也很正常啊

一个lun对应多少个路径的查看命令是multipath -l
8个多路径/dev/sdb~/dev/sdi对应同一个wwid
[root@bidg ~]# multipath -l
3600a098038303867635d4a48624e5466 dm-0 NETAPP,LUN C-Mode
size=1.1T features='3 queue_if_no_path pg_init_retries 50' hwhandler='0' wp=rw
|-+- policy='round-robin 0' prio=0 status=active
| |- 11:0:0:0 sdb 8:16  active undef running
| |- 12:0:0:0 sdf 8:80  active undef running
| |- 11:0:3:0 sde 8:64  active undef running
| `- 12:0:1:0 sdg 8:96  active undef running
`-+- policy='round-robin 0' prio=0 status=enabled
  |- 11:0:2:0 sdd 8:48  active undef running
  |- 12:0:2:0 sdh 8:112 active undef running
  |- 11:0:1:0 sdc 8:32  active undef running
  `- 12:0:3:0 sdi 8:128 active undef running



这个lun对应的8个多路径/dev/sdb~/dev/sdi对应的uuid一样；

这个lun对应的8个多路径/dev/sdb~/dev/sdi对应的wwid一样

5、UDEV
UDEV是Linux提供的一种让用户对设备进行自定义命名的机制。可以通过UDEV将WWID/UUID信息跟磁盘路径映射起来，这样也可以保证设备路径永久不变。

cat /etc/sysconfig/network-scripts/ifcfg-bond0
