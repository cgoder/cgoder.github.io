---
title: etcd and MQTT cluster
date: 2016-07-25 15:47:38
categories:
 - study
tags:
 - mqtt
 - cluster
 - etcd
---

看了下[ectd](https://github.com/coreos/etcd)的介绍，发现这个数据库与MQTT协议的mosqutto一起搭简易的mqtt cluster简直是绝配。
主要是ectd的设计宗旨符合这个需求。主要如下：
- 集群化：天生就带有分布式集群特性。
- 同步性：天生就支持分布式数据同步特性。
- 键的层次性：键的组织结构像mqtt协议的topic一样具有路径属性。
- watch功能：像mqtt协议的P/S一样，可以监控某个键值变化。并支持像定时器一样，监测到变化后执行某个动作(exec-wathc)。

目前看到的特性就感觉非常不错。其它的有待继续深入了解。。。
