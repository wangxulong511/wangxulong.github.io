# Warning: VKTM detected a time drift.

```
集群版本：11.2.0.4  数据库版本：11.2.0.4 

文章来自：https://www.linuxidc.com/Linux/2016-03/128903.htm

时间是包括数据库系统在内的诸多信息系统基础件的重要因素。对于运行在操作系统OS之上的中间件组件而言，获取到一个准确、连续和一致的时间非常重要，特别是多节点的环境下。如果没有一个统一的时间管理机制，其上的cluster组件工作是及其困难的。

本篇主要介绍Oracle vktm时间后台进程报警的Bug问题。

1、从11g VKTM进程谈起

对Oracle数据库，避免对于操作系统层面时间的调用，维持一个统一稳定的时间体系一直是发展方向。在11g中，一个独立的后台进程vktm被引入到实例体系下。

VKTM进程全称为“Virtual Keeper of Time Process”，用于给数据库运行和间隔运算计量提供出一个统一的时间服务。官方解释是：

VKTM acts as a time publisher for an Oracle instance. VKTM publishes two sets of time: a wall clock time using a seconds interval and a higher resolution time (which is not wall clock time) for interval measurements. The VKTM timer service centralizes time tracking and offloads multiple timer calls from other clients.

在11g之前的版本中，如果数据库实例（包括ASM和RAC Instance）需要当前时间的时候，都调用操作系统层面的时间获取函数（例如：gettimeofday()）。进入11g之后，这个动作就由统一的VKTM负责完成，并且在进程内部保留时间过程。其他进程如果需要时间，都通过这个进程间接获得。专门的时间后台进程的出现，最直接的效果就是减少了同操作系统内核的交互，提高了性能。

2、Time Drift问题

在一次日志巡检中，笔者发现alert log中存在如下的报警信息：

Tue Dec 22 21:56:26 2015

Warning: VKTM detected a time drift.

Time drifts can result in an unexpected behavior such as time-outs. Please check trace file for more details.

Tue Dec 22 23:02:44 2015

Warning: VKTM detected a time drift.

Time drifts can result in an unexpected behavior such as time-outs. Please check trace file for more details.

alert log是我们发现数据库运行问题最直接的途径和方法。从11g开始，一些预测、诊断性的信息，也都通过alert log进行输出，期望实现数据库故障问题预先诊断。

从日志信息看，数据库在两个时间点经历了两次VKTM进程的Time Drift现象。Drift是漂移、浮动的含义。VKTM作为时间维护后台进程，在两个时间点中似乎发生了快速的前移和后退。

根据日志中信息时间信息，我们查找定位故障发生时的trace文件。

[oracle@vLIFE-URE-PRD-DB-PRIMARY trace]$ ls -l | grep vktm

（篇幅原因，有省略……）

-rw-r-----. 1 oracle oinstall    90 Dec 21 14:48 UREPRD_vktm_22700.trm

-rw-r-----. 1 oracle oinstall  1846 Dec 22 23:02 UREPRD_vktm_23138.trc

-rw-r-----. 1 oracle oinstall    128 Dec 22 23:02 UREPRD_vktm_23138.trm

对应的Trace文件信息如下：

kstmmainvktm: failed in setting elevated priority

Verify: SETUID is set on ORADISM and restart the instance

highres_enabled

VKTM running at (100ms) precision 

kstmrmtickcntkeeper: param _dbrm_quantum will not be effective

*** 2015-12-21 16:15:27.883

[Start] HighResTick = 1450685727883070

kstmrmtickcnt = 0, ksudbrmseccnt[0] = 1450685727

kstmchkdrift (kstmrmtickcntkeeper:highres): Time jumped forward by (16898372)usec at (1450792586464465) whereas (1000000) is allowed

*** 2015-12-22 21:56:27.439

kstmchkdrift (kstmrmtickcntkeeper:lowres): Time jumped forward by (18000000)usec at (1450792587) whereas (5000000) is allowed

kstmchkdrift (kstmrmtickcntkeeper:highres): Time jumped forward by (1112648)usec at (1450796564826494) whereas (1000000) is allowed

在alert结果中，我们看到两次漂移drift的现象。信息中，我们看到一个单位usec，折合0.000001秒钟。那么，两个提示的意思就比较明确了：VKTM进程向前分别跳跃了18秒和1.1秒。规定的跳跃时间是1秒钟。

那么，下一个问题是，问题是怎么产生的？我们是否需要介入处理？alter log中一些告警信息是基于数据库诊断规则，警告的内容涵盖内存、安全、存储等多个方面。并不是每一种信息都需要进行处理，很多信息内容都是基于最佳Oracle工作实践的一种建议。

在MOS中，笔者定位到了一篇针对alert log中出现Time Drift提示错误的文章，名称为：Time Drift Detected. Please Check VKTM Trace File for More Details，文章编号：ID 1347586.1。

在文章中，Oracle认为这个错误在11.2.0.2到11.2.0.3之间会不断地出现，已经在11.2.0.4版本中被修复。Bug编号为：11837095 "TIME DRIFT DETECTED" APPEARS INTERMITTENTLY IN ALERT LOG, THO' EVENT 10795 SET。

如果需要对该Bug进行单独修复，需要进行补丁patch 11837095操作。之后通过设置等待事件10795来进行控制。

alter system set event="10795 trace name context forever, level 2" scope=spfile;

这种错误的潜在影响，MOS文章中的解释如下：

Impact of the error:

The time drifts usually occurring less than 1sec and 5 sec for forward and backward respectively are permissible and OK.

If the traces are emitting time drifts of amount beyond these ranges, then it needs to be analyzed.

Most of the times, during high loads, there would be issues with underlying OS due to virtual memory, network time protocol improper configuration etc.

In general VKTM process need to be scheduled in every 10ms, if due to above reasons this is not happening we see the time drifts and to certain level (mentioned above) are permissible.

Eventually, this probably would cause the resource manager to take improper decisions and can lead to a hang in worst case.

VKTM process trace file can be found under bdump, However in this case the trace file doesn't contain useful information, Which makes the message ambiguous.

大部分情况下，由于系统高负载或者内存调度配置有问题，VKTM的每10ms作业会有问题，可能会出现1-5秒的Drift漂移。如果这种现象出现很频繁，就需要整体考虑操作系统和数据库配置问题。VKTM出现漂移，可能会影响到resource manager的工作情况。

3、结论

VKTM是Oracle 11g中新引入的一个后台进程。我们在面对alert log中出现的各种奇怪问题的时候，最好主动咨询一下官方MOS，看看是否有相似的方案解决。小问题尽早解决，才能保证系统不出现大故障。

 ```
 
 
