---
title: shadowsock+proxifier
date: 2016-09-08 14:33:49
categories:
 - life
tags:
 - vpn
 - shadowsocks
---

如果愉快地全局爬梯？这是个问题！
<!-- more -->

[shadowsocks](https://github.com/shadowsocks)这个科学上网神器不用多说，这个小飞机图标，程序员都应该知道吧？（不知道你还好意思写代码？）
![](http://ww1.sinaimg.cn/large/772d7a33gw1f7m7ip2iw1j205k05kmx8.jpg)
但是这家伙只能代理浏览器，并不能代理本地应用。比如你要下个Android Studio的sdk，那就呵呵了。所以，这个时候就需要系统全局性cross the wall。
怎么办？这时候就要另外一个老家伙神器[proxifier](https://www.proxifier.com/)了。
![](http://ww3.sinaimg.cn/large/772d7a33gw1f7m7l4aefdj207306bjrx.jpg)
> 这家伙好老了，记得2003年还在用SIEMENS C55手机加上移动wap无限流量卡出差报连接GPRS拔号上网时，就用到了它来做代理。基本原理就是通过把http请求转发代理到中国移动的10.0.0.172的80端口去，以实现所有的连接都是通过移动的wap网（这货绝对是时代的怪胎）上去。这个移动wap无限流量的手机卡我至今仍然在当主力号码用，伴我度过青春年代的SIEMENS虽然没了，那个C55,还有后来的M65,S65都仍然还在抽屉的角落里躺着。。。
到了现在这个4G的时代，不得不再次祭出proxifier这个神器，全然因为GFW的存在。不知道GFW在多年之后会不会成为像wap/tdscdma之类的技术怪胎一样“流芳万世”。


下载proxifier，地址[*来自汉化新世纪*](http://www.hanzify.org/software/13717.html)。
## 添加代理地址
打开proxifier程序，选择菜单栏的`配置文件`->`代理服务器`，添加一个新的代理：
+ 地址：127.0.0.1 (shadowsocks应用的地址)
+ 端口：1080 (shadowsocks应用的本地端口)
+ 类型：socks5
![](http://ww4.sinaimg.cn/large/772d7a33gw1f7m7ej013bj20cb071wf6.jpg)

## 添加代理规则
选择菜单栏的`配置文件`->`代理规则`来设置自己想要走代理的应用程序。如图所示，添加了一个代理
![](http://ww2.sinaimg.cn/large/772d7a33gw1f7m7saephgj20db0dljsk.jpg)
最后代理规则可能如下图
![](http://ww3.sinaimg.cn/large/772d7a33gw1f7m7ppusejj20jy047gme.jpg)

## 设置DNS
这个时候其实就可以让代理规则中的应用走shadowsocks代理了。但是GFW内可是有很严重的DNS污染的。所以我们还要把DNS设置一下，以防止被各种污染劫持。
选择菜单栏的`配置文件`->`名称解析`，选择`通过代理解析主机名称`项
![](http://ww3.sinaimg.cn/large/772d7a33gw1f7m7vfvsu2j20az0c5jse.jpg)

这时候就OK了，其它的功能项可以等用得着的时候再google一下吧。


## 参考文献 
[使用Proxifier把shadowsocks代理转为真·全局](http://www.dou-bi.com/ss-jc7/)
