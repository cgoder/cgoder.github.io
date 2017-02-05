---
title: openwrt opkg architecture error
date: 2016-09-26 14:03:02
categories:
 - study
tags:
 - Openwrt
---

opkg出错为什么？
<!-- more -->

在折腾coocaa智能路由器的过程中，碰到过一个问题。使用`opkg update`命令时出现如下示例的错误。
```shell
Package XXX version XXXX has no valid architecture, ignoring.
```
google一下，原来是由于cpu架构不同导致的与源冲突。
coocaa这个路由器是MT7628的cpu，我用的Pandora固件的后缀是ralink，可以用ramips架构的包。所以就把opkg的配置文件改了下，文件路径是`/etc/opkg.conf`。
```shell
[root@PandoraBox:/root]#cat /etc/opkg.conf
dest root /
dest ram /tmp
lists_dir ext /var/opkg-lists
option overlay_root /overlay
src/gz base https://downloads.openwrt.org/snapshots/trunk/ramips/mt7628/packages/base
src/gz packages https://downloads.openwrt.org/snapshots/trunk/ramips/mt7628/packages/packages
src/gz luci https://downloads.openwrt.org/snapshots/trunk/ramips/mt7628/packages/luci
arch all 100
arch ralink 200
arch ramips 300
arch ramips_24kec 400
```

`arch`字段后面跟着的是平台架构名，平台名后面跟的数字代表优先级。具体含义可参见[openwrt wiki](https://wiki.openwrt.org/doc/techref/opkg#configuration)
> Adjust Architectures

> By default, opkg only allows packages with the architecture all (= architecture independant) and the architecture of the installed target. In order to source packages from a foreign, but compatible target, the list of allowed architectures can be overrided in opkg.conf with the use of arch options:
>> arch all 100
>> arch brcm4716 200
>> arch brcm47xx 300

> This example would allow installing brcm47xx (= family of SoCs) packages on the brcm4716 (a specific SoC) target. The number specifies a priority index which is used by opkg to determine which package to prefer in case it is available in multiple architectures.
