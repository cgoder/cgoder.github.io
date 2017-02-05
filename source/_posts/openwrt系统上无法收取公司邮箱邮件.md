---
title: openwrt系统上无法收取公司邮箱邮件
date: 2016-10-20 09:59:56
categories:
 - study
tags:
 - Openwrt
---

Openwrt小坑不断！
<!-- more -->

用基于openwrt的路由器在公司使用时，发现不能正常收发公司邮箱的邮件（用的thundbird），google了一下，好像是dnsmasq的问题。
> 如果在dnsmasq.conf中加上 “stop-dns-rebind” 就可能导致这个问题，去掉这个选项，重启就好了。去掉这个选项是在/etc/config/dhcp里面，修改`option rebind_protection ‘0’`。直接修改dnsmasq.conf会被覆盖掉。
