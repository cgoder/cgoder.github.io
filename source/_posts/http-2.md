---
title: HTTP/2多路复用
date: 2016-08-03 10:35:57
categories:
 - study
tags:
 - http2
---

很浅显的解释一下http2的多路复用。
<!-- more -->
学习HTTP/2的过程中，发现其实大部分基础协议的原理和实现其实都是共通的。比如近来大热的HTTP/2协议与熟知的DVB协议。两者在基础原理上的共通是，都应用了`多路复用`的概念。使得一个TS传输流(DVB)/TCP连接(HTTP2)里可包含多个同通道的数据。DVB协议里是不同的PES数据（可以表义理解为一个节目数据）；HTTP2协议里是不同的meta数据。

## 多路复用
+ 第一路数据通道
![](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-justin.jpg)
+ 第二路数据通道
![](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-ikea.jpg)
+ 多路数据复用后通道
![](https://raw.githubusercontent.com/bagder/http2-explained/master/images/train-multiplexed.jpg)
