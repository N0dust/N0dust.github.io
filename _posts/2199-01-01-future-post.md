---
title: 'Change Steams集群同步延迟'
date: 2024-09-01
tags:
  - cool posts
  - mongo
---

### 背景
腾讯的Mongo集群，共三个节点。页面有一个仅读从节点链接：
```
mongodb://userName:password@192.168.1.1:27017,192.168.1.2:27017/test?replicaSet=cmgo-5xvzuq9x_0&authSource=admin&readPreference=secondaryPreferred
```

### 问题
直接用这个链接去读Steams，捕捉到数据变动后立马再用此链接去读数据，有概率会读到变更之前的数据。

 举个例子：
 已数据: `{_id:"A",number:2}`
 steams出消息：id为A的number变更为1。收到消息有立刻去查这条数据的number，预期结果是1，但有可能是2。

我将其归因为集群间同步延时问题，这也导致该问题不好复现，毕竟延迟间隙不会总能被捕捉到。

### 解决方案
盯着单个从节点，读流和数据查询都从这来。剥离集群同步延迟的影响，靠的是最终一致性。
如：
```
mongodb://userName:password@192.168.1.2:27017/admin
```
缺点是读压力会偏到一个节点上，具体要看业务性能压力。
