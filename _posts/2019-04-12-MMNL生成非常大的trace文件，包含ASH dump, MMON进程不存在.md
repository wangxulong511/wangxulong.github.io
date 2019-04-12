---
layout:     post
title:      MMNL生成非常大的trace文件，包含ASH dump, MMON进程不存在
subtitle:   Oracle trace mmon
date:       2019-04-12
author:     静观知心
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - oracle
    - mmon
    - trace
---

# MMNL生成非常大的trace文件，包含ASH dump, MMON进程不存在

> http://ju.outofmemory.cn/entry/367557

Oracle 11.2.4 RAC trace 日志暴增

trace 日志中报错如下
```
*** 2019-04-12 09:33:38.346
kewa_sampler: Re-signalling MMON(237) (!= 212) at 12-APR-19 09.33.38.346 AM
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453

*** 2019-04-12 09:33:41.373
kewa_sampler: Re-signalling MMON(249) (!= 237) at 12-APR-19 09.33.41.373 AM
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453

*** 2019-04-12 09:33:51.411
kewa_sampler: Re-signalling MMON(30) (!= 249) at 12-APR-19 09.33.51.411 AM
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453

*** 2019-04-12 09:33:54.440
kewa_sampler: Re-signalling MMON(41) (!= 30) at 12-APR-19 09.33.54.440 AM
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453

*** 2019-04-12 09:34:06.474
kewa_sampler: Re-signalling MMON(56) (!= 41) at 12-APR-19 09.34.06.474 AM
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453

*** 2019-04-12 09:35:35.986
kewa_sampler: Re-signalling MMON(91) (!= 56) at 12-APR-19 09.35.35.986 AM
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453
kewa_sampler_cb: Exception handler 453
kewa_sampler_cb: Cleared error 453

```

查看进程，发现mmon进程不在，解决办法是需要重启实例
