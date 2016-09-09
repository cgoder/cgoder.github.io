---
title: perfect proxifier+UWP
date: 2016-09-09 09:06:59
categories:
 - life
tags:
 - vpn
 - shadowsocks
---
前面的一篇文章{% post_link shadowsock-proxifier shadowsock+proxifier %}简单说了下如何把shadowsocks利用到系统全局代理的方法。
但是这个方法有个bug是，装完proxifier之后，所有的windows 10的metro应用都不能用了。比如UMP神软`Hello,TV`。那怎么办呢？Google一下，原理是metro应用和系统其它应用不一样，不走本地代理，所以当proxifier安装后，流量都走127.0.0.1，但是metro不走。所以，只要安装了proxifier软件，不管你有没有打开使用它，metro应用都不能联网。
找了好长时间，google出来都是利用Fiddler软件里的一个叫AppContainer LoopBack的组件，使metro应用能够走本地127.0.0.1的代理来实现。
昨天突然灵光一现，想到了一个方法。既然不安装proxifier就是正常的，那我不安装不就行了嘛。不过不安装proxifier，那就不能做全局代理了啊。嘿嘿，别忘了[proxifier官网](https://www.proxifier.com/download.htm)可是提供portable版本的下载，这个版本不用安装也能使用。于是下载了proxifier portable版本来试了一下，果然可行！配置文件把要代理的应用加上就行了。要用的时候打开proxifier，不用的时候关掉。
完美！

PS：提供个可用的proxifier portable版本注册码`L6Z8A-XY2J4-BTZ3P-ZZ7DF-A2Q9C`
