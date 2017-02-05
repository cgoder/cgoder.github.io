---
title: COOCAA SWR100K to Pandora firmware
date: 2016-09-26 14:41:06
categories:
 - study
tags:
 - Openwrt
 - MTK
---

最近在折腾把手上的酷开智能路由器给刷成原版的openwrt。
<!-- more -->

> coocaa智能路由器分realtek和mtk两种版本。本文仅指型号为SWR100K的MTK芯片版本。硬件配置是MT7628AN+128MB+8M。

就是这货 >>> 外观
![](http://ww4.sinaimg.cn/large/772d7a33gw1f86yfypwh1j20aj0fvaa5.jpg)
![](http://ww1.sinaimg.cn/large/772d7a33gw1f86yg40uppj20bs0h5glr.jpg)
![](http://ww3.sinaimg.cn/large/772d7a33gw1f86ygab0wqj20fe0gut94.jpg)
这家伙的原版固件其实也能用。但是不能翻墙，把openwrt自带的东西改了好多，跟小米一个德性。而且还改得不好（不用问我怎么知道的）。

> 可以通过telnet或者ssh连接后，在命令行下输入`opkg print-architecture | awk '{print $2}'`命令来查看cpu架构。

<!-- more -->

## 刷不死boot-Breed
首先把它的boot给换了。换成[breed](http://www.right.com.cn/forum/forum.php?mod=viewthread&tid=161906)，以防刷死。选择下载`breed-mt7688-reset38.bin`这个文件。
- 上电的同时按住reset/wps键，等前面指示灯狂闪成（紫色）时松手。
- 在cmd命令行下，输入`ping 192.168.1.1 -t`看能不能ping通。记得电脑和路由器的lan口连接！
- 如果能ping通，但是上不了网。那么好了，此时你就在恢复模式(姑且这么叫吧)下。
- 此时，可以在cmd里用tftp命令传输文件给路由器来改写uboot。如`tftp -i 192.168.1.1 put D:\breed-mt7688-reset38.bin`.
- 写的过程中可能一直没结束的动静，你以为死了？挂了？其实不然，它成功了。breed很小的，十来秒就搞定了。其实你就可以重启上电了。
- 此时，Breed已经替换原来的uboot了。系统也是能正常进入的。

ps:另外还有一种替换uboot成breed的方法。是模拟mtd命令写的方式做的。具体可以google一下mtd写固件的方法。

## 进入Breed更新固件
怎么用breed？hackpascal大神提供了2种方法，我个人更推荐`Breed Enter工具`的方法来，简单易懂。就是打开`Breed Enter工具`这个小软件，点击开始监测，然后路由器上电，如果软件监测到breed启动了，就提示一下，并中断路由器的启动流程。原理很简单，估计就是大神在breed里自己加了个私有协议，小软件通过与breed里的私有协议通信来达到监测的目的，并实现监测到了breed，就让breed不继续init路由器kernel/system的目的。
当小工具软件监测到后，就可以在浏览器里打开`192.168.1.1`的网址，这时就可以看到breed的界面了。后面按照breed的操作指南去做就行了。其实界面很简单，也容易理解，这里就不赘述了。

## 选择固件
网上mt7628的固件其实不多，而且这个路由器的flash只有8MB（其实是16MB，不要问我怎么知道的）可用。没办法，可选的不多，除非你自己去编，更麻烦。
看了下，有[不死鸟固件](http://www.right.com.cn/forum/forum.php?mod=viewthread&tid=192166)和[潘多拉固件](http://www.right.com.cn/forum/forum.php?mod=viewthread&tid=165986)。俩者之中更信任Prandora box，就选择它了。
不过这里都没有集成shadowsocks(新版pandora集成了，不过固件太大)模块，没办法，自己上吧，自己安装。

## 更新固件
这就不写了。在breed里看一下就知道怎么更新了。
提醒下，固件不要超过8MB。比如新的Pandora固件就超过10MB。


## 添加Shadowsocks/ChinaDNS
添加ss有多种方法。我选的最简便的一种，直接更换包源。因为openwrt snapshot源里有mt7628平台，并且包括了shadowsocks包，只不过chinadns的没有，要另外弄。

- 先更换源。
```shell
[root@PandoraBox:/root]#cat /etc/opkg.conf 
dest root /
dest ram /tmp
lists_dir ext /var/opkg-lists
option overlay_root /overlay
src/gz base http://downloads.openwrt.org/chaos_calmer/15.05.1/ramips/mt7628/packages/base
src/gz packages http://downloads.openwrt.org/chaos_calmer/15.05.1/ramips/mt7628/packages/packages
src/gz luci http://downloads.openwrt.org/chaos_calmer/15.05.1/ramips/mt7628/packages/luci
src/gz openwrt_dist http://openwrt-dist.sourceforge.net/dist/base/ramips
src/gz openwrt_dist_luci http://openwrt-dist.sourceforge.net/dist/luci
arch all 100
arch ralink 200
arch ramips_24kec 300
arch ramips 400
[root@PandoraBox:/root]#
```
最上面的源是换成CC的最新源，为了依赖包的最新。最后2个源是shadowsocks的官方下载源。后面的Arch字段为了匹配平台。

配置完之后就可以直接执行命令了
```shell
opkg update
opkg install ChinaDNS
opkg install luci-app-chinadns
opkg install dns-forwarder
opkg install luci-app-dns-forwarder
opkg install redsocks2
opkg install luci-app-redsocks2
opkg install shadowsocks-libev
opkg install luci-app-shadowsocks
opkg install ShadowVPN
opkg install luci-app-shadowvpn
```

完成之后配置一下就OK了。

也可以懒点，直接用脚本
```shell
wget http://openwrt-dist.sourceforge.net/auto_install.sh
chmod +x auto_install.sh
./auto_install.sh
```

## 配置注意事项
1. 在luci-app-shadowsocks的配置页面`访问控制`里，要把`内网区域`里的`代理自身`选择**正常代理**，`代理类型`选择**正常代理**。

2. chinaDNS周期更新可以使用如下命令：
> wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/chinadns_chnroute.txt

3. 另外，要把`网络`->`DHCP/DNS`->`基本设置`菜单下的`本地服务器`填上chinadns的代理端口`127.0.0.1#5353`，同时把`Host和解析文件`菜单下的`忽略解析文件勾选`。


enjoy!

## 参考文献 
[http://openwrt-dist.sourceforge.net/](http://openwrt-dist.sourceforge.net/)
[https://sourceforge.net/p/openwrt-dist/wiki/Home/](https://sourceforge.net/p/openwrt-dist/wiki/Home/)
[https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks)
[https://softwaredownload.gitbooks.io/openwrt-fanqiang](https://softwaredownload.gitbooks.io/openwrt-fanqiang)
