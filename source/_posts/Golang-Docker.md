---
title: Golang@Docker
date: 2016-07-20 13:55:14
categories:
 - study
tags:
 - golang
 - Docker
---

## 学习资料
前几天花点时间沉下心来学习了下[Docker](https://www.docker.com/)。觉得这东西真不错。就赶紧把golang在Docker上应用了一把。虽然在[docker hub](https://hub.docker.com/)上已经有很多golang的镜像了，但是基本上都是基于ubuntu的，而ubuntu体积实在太大。即使仅用ubuntu的纯粹文件系统做出来的镜像都已经是1.1GB了。所以以就找了个mini型的Linux发行版本[Alpine Linux](http://alpinelinux.org/)，做了个基于Alpine的镜像。其中golang的版本是1.6.2。
- docker hub上的下载页面地址在[**这里**](https://hub.docker.com/r/gcoder/golang/)。（由于docker hub在墙外，所以速度很慢）
- aliyun上的下载页面地址在[**这里**](https://dev.aliyun.com/detail.html?repoId=9014)。

主要的参考资料来自于gitbook上的一本书[《Docker--从入书到实战》](https://www.gitbook.com/book/yeasy/docker_practice/details)，这本书讲得挺浅显易懂的。所以也顺便推荐下这本书。
![](https://ek8whxe.cloudimg.io/s/cdn/x/https://www.gitbook.com/content/book/yeasy/docker_practice/docker_primer.png?v=15.2.1)

Docker深入一些的概念和原理来自于[酷壳](http://coolshell.cn/?s=docker)的文章。相对来说，[酷壳](http://coolshell.cn/?s=docker)的文章就要有比较深入的Linux知识才能真正看得懂了。不过慢慢啃还是能啃下去的。比如下面这张Docker分层原理图，我就得看一会儿才明白。
![](http://coolshell.cn//wp-content/uploads/2015/08/docker-filesystems-busyboxrw.png)

<!-- more -->

## 简单过程
学习的东西就以上这些。下面说下基本的构建过程，其实很简单，就4步：
1. 下载Alipne文件系统包。
2. 将Alipne文件系统包导入Docker镜像
3. 运行Docker镜像安装golang环境
4. 将已经安装好golang环境的Docker镜像导出成新的镜像。

其中，第3、4步可以合并成一个步骤，就是直接写个Dockerfile(类似Makefile，用于指导Docker程序如何编译镜像的配置文件)，由Dockerfile自己编译出新的Docker镜像即可。

下面，我们来实战一下。以Docker hub上的官方Alpine镜像为基础。

## 实战

### 下载Alpine镜像
首先，从docker hub上下载官方的[Alpine镜像](https://hub.docker.com/_/alpine/)。下载命令如下：（当然，你也可以自己下载Alpine文件系统自己用Dockerfile编译制作）
```shell
9527@ubuntu:~/Docker/alpine$ sudo docker pull alpine
Using default tag: latest
latest: Pulling from library/alpine
Digest: sha256:3dcdb92d7432d56604d4545cbd324b14e647b313626d99b889d0626de158f73a
Status: Downloaded newer image for alpine:latest
9527@ubuntu:~/Docker/alpine$
```

现在，我们来看一下是不是已经下载下来了。用docker images指令
```shell
9527@ubuntu:~/Docker/alpine$ sudo docker images 
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
golang                                     1.6.2               5101f182bc8c        6 days ago          192.1 MB
ubuntu64                                   golang              4590d859a6ff        6 days ago          1.055 GB
ubuntu64                                   16.04               19ce1c3d99ca        7 days ago          500.1 MB
alpine                                     latest              4e38e38c8ce0        3 weeks ago         4.799 MB
9527@ubuntu:~/Docker/alpine$
```
看一下，才不到4.8MB，好mini有木有？看看它楼上的ubuntu 16.04 x64的rootfs镜像，真是小几个数量级啊。怪不得Docker官方基础镜像要准备抛弃ubuntu，投入Alpine的怀抱了。

### 写Dockerfile
写个Dockerfile，把需要做的事情按步骤写好。内容如下：
```shell
9527@ubuntu:~/Docker/alpine$ cat Dockerfile
FROM alpine:latest
RUN apk --update add go
CMD /bin/ash
9527@ubuntu:~/Docker/alpine$
```

### 生成新镜像
```shell
9527@ubuntu:~/Docker/alpine$ sudo docker build -t="golang:1.6.2" .
```
指令敲过之后，就是漫长的等待。因为要更新，要下载golang包，配置环境，然后打包。最终生成的镜像如下：
```shell
9527@ubuntu:~/Docker/alpine$ sudo docker images 
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
golang                                     1.6.2               5101f182bc8c        6 days ago          192.1 MB
```
最终生成的镜像有192MB之多。怪不得ubuntu上安装golang，体积能从550MB急剧增加到1.1GB了。

### 试运行
好了，现在我们的镜像已经生成成功了。简单吧？那么，我们现在来试运行一下
```shell
9527@ubuntu:~/Docker/alpine$ sudo docker run -it golang:1.6.2
[sudo] password for 9527: 
/ # ls
bin      dev      etc      home     lib      linuxrc  media    mnt      proc     root     run      sbin     srv      sys      tmp      usr      var
/ # go version
go version go1.6.2 linux/amd64
/ # 
```
很明显进入了linux shell，打印看看golang的版本，是正确的。

### hello world
那我们接下来试着写个hellworld试试。
```shell
/ # vi hello.go
/ # 
/ # cat hello.go 
packge main

import "fmt"

func main(){
    fmt.Println("Hello Golang!")
}
```
写好了，我们来运行一下试试。
```shell
/ # go run hello.go 
Hello Golang!
/ # 
```
OK，可以了，打完收工！
