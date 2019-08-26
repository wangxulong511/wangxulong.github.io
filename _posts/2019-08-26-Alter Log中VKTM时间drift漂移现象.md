# Alter Log中VKTM时间drift漂移现象
> https://www.linuxidc.com/Linux/2016-03/128903.htm

在巡检时发现RAC ASM 的 alter内容如下,VKTM
```
Time drifts can result in an unexpected behavior such as time-outs. Please check trace file for more details.
Mon Aug 05 09:35:48 2019
Warning: VKTM detected a time drift.
Time drifts can result in an unexpected behavior such as time-outs. Please check trace file for more details.
Mon Aug 19 05:51:58 2019
Warning: VKTM detected a time drift.
Time drifts can result in an unexpected behavior such as time-outs. Please check trace file for more details.
Thu Aug 22 05:52:10 2019
Warning: VKTM detected a time drift.
Time drifts can result in an unexpected behavior such as time-outs. Please check trace file for more details.
```
