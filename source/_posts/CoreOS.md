---
title: CoreOS
date: 2016-07-26 15:20:06
categories:
 - study
tags:
 - CoreOS
---

CoreOS是个什么东西？
<!-- more -->
## 简介
[CoreOS](https://coreos.com/) 源自Google ChromeOS。是一个全新的、面向数据中心设计的 Linux 操作系统。
CoreOS 专门针对大型数据中心而设计，旨在以轻量的系统架构和灵活的应用程序部署能力简化数据中心的维护成本和复杂度。从一开始就决定了CoreOS更加适合应用于一个集群环境而不是一个传统的服务器操作系统。
![](http://ww1.sinaimg.cn/large/772d7a33jw1f669ooyc84j21650fzgnk.jpg)
> CoreOS 没有提供包管理工具，而是通过容器化 (containerized) 的运算环境向应用程序提供运算资源。应用程序之间共享系统内核和资源，但是彼此之间又互不可见。这样就意味着应用程序将不会再被直接安装到操作系统中，而是通过Docker运行在容器中。这种方式使得操作系统、应用程序及运行环境之间的耦合度大大降低。
> CoreOS 采用双系统分区 (dual root partition) 设计。两个分区分别被设置成主动模式和被动模式并在系统运行期间各司其职。主动分区负责系统运行，被动分区负责系统升级。一旦新版本的操作系统被发布，一个完整的系统文件将被下载至被动分区，并在系统下一次重启时从新版本分区启动，原来的被动分区将切换为主动分区，而之前的主动分区则被切换为被动分区，两个分区扮演的角色将相互对调。同时在系统运行期间系统分区被设置成只读状态，这样也确保了CoreOS的安全性。CoreOS的升级过程在默认条件下将自动完成，并且通过cgroup对升级过程中使用到的网络和磁盘资源进行限制，将系统升级所带来的影响降至最低。

*`注：CoreOS原来是Docker的基础。不过可能觉得Docker现在越来越不pure，越来越背离初衷？于是现在自己另起炉灶，搞了个与Docker类似的东西`[Rkt](https://github.com/coreos/rkt)*

## CoreOS架构
* 从整体架构来看，CoreOS就是一个极简的，只跑Docker的Linux系统。
* 从传统分层来看，CoreOS像是底层硬件+固件，Docker像是上层应用层。
* 从软件分层来看，CoreOS像是操作系统,Docker+etcd像是内核，Docke容器像是进程。
* 从对比模式来看，CoreOS像是hypervisor，Docker容器像是虚拟机。

<!-- more -->

> CoreOS 内置了两个服务：etcd 和 fleet。它们都是CoreOS的子项目。etcd是一个高可用的键值存储系统，主要用于共享配置和服务发现，类似于 ZooKeeper 和 Doozer。 fleet是一个分布式的container发布工具，用于进行集群中任务的提交和管理。可以这么理解，etcd用来自动化构建CoreOS集群，而fleet则是运行于CoreOS集群之上的任务（docker）管理平台。也就是说，CoreOS设计之初，就将运行环境定位于集群&平台。

![](http://ww4.sinaimg.cn/large/772d7a33gw1f668nmagltj20cs0a6752.jpg)
![](http://ww1.sinaimg.cn/large/772d7a33jw1f669sb8tppj20hq09z75a.jpg)

## etcd
etcd在CoreOS中的位置：
![](http://ww1.sinaimg.cn/large/772d7a33gw1f668mufzjoj20dw08pt9r.jpg)
> **在CoreOS 集群中处于骨架地位的是etcd。**etcd是一个分布式 key/value存储服务，CoreOS集群中的程序和服务可以通过etcd共享信息或做服务发现。etcd基于非常著名的raft一致性算法：通过选举形式在服务器之中选举Lead来同步数据，并以此确保集群之内信息始终一致和可用。etcd以默认的形式安装于每个CoreOS系统之中。在默认的配置下，etcd使用系统中的两个端口：4001和7001，其中4001提供给外部应用程序以HTTP+Json的形式读写数据，而7001则用作在每个etcd之间进行数据同步。用户更可以通过配置CA Cert让 etcd以HTTPS的方式读写及同步数据，进一步确保数据信息的安全性。


## 参考文献 
[http://www.infoq.com/cn/articles/what-is-coreos](http://www.infoq.com/cn/articles/what-is-coreos)
[https://www.airpair.com/coreos/posts/coreos-with-docker](https://www.airpair.com/coreos/posts/coreos-with-docker)
[http://cloud.51cto.com/art/201501/464025_all.htm](http://cloud.51cto.com/art/201501/464025_all.htm)
[《CoreOS实践指南》](http://www.jianshu.com/p/da7ff503064c)
[《CoreOS实战》](http://www.infoq.com/cn/CoreOSAction)
