---
title: docker源码解读
date: 2017-03-21 17:07:14
categories:
 - study
tags:
 - golang
 - Docker
---
值得备份的一系列docker源码解读文章。
<!-- more -->

作者是daocloud的合作创始人，大神。
解读基于docker 1.2源码 [https://github.com/docker/docker/tree/v1.2.0](https://github.com/docker/docker/tree/v1.2.0)

系列文章如下：
[Docker源码分析（一）：Docker架构](http://lib.csdn.net/article/docker/922)
[Docker源码分析（二）：Docker Client创建与命令执行](http://lib.csdn.net/article/docker/927)
[Docker源码分析（三）：Docker Daemon的启动](http://lib.csdn.net/article/docker/1071)
[Docker源码分析（四）：Docker Daemon之NewDaemon实现](http://lib.csdn.net/article/docker/923)
[Docker源码分析（五）：Docker Server的创建](http://lib.csdn.net/article/docker/926)
[Docker源码分析（六）：DOCKER DAEMON网络](http://lib.csdn.net/article/docker/925)
[Docker源码分析（七）：Docker Container网络 （上）](http://lib.csdn.net/article/docker/924)

> 其中有一点涉及到golang的init函数的说明，可能值得商榷。源码解析文章中说golang中同一package的不同文件的init函数调用顺序不确定。**但是有证据表明，在同一package的不同init函数调用顺序是按照首字母顺序来确定的。*** 当然，init函数还是按照package被import的顺序来逆序调用的，且在main函数之前。
