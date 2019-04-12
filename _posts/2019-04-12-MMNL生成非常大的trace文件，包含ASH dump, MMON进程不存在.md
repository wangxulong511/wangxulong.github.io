# MMNL生成非常大的trace文件，包含ASH dump, MMON进程不存在

http://ju.outofmemory.cn/entry/367557

Oracle 11.2.4 RAC trace 日志暴增

trace 日志中报错如下
```
*** 2019-04-12 08:23:57.349
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453

```

查看进程，发现mmon进程不在，解决办法是需要重启实例
