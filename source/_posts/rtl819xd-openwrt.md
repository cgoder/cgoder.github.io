---
title: rtl819xd openwrt
date: 2016-09-29 13:55:03
categories:
 - study
tags:
 - Openwrt
 - realtek
---

手残搞死人，
折腾无极限！
<!-- more -->

基于MTK芯片MT7628的酷开路由器折腾好了之后，手热难耐，继续折腾扯淡的RealTek芯片RTL8197DL的酷开路由器。俩者外观配置并无二致，只是主芯片和无线芯片不同而已。型号好像都是一样的。。。

不过号称最坑爹的RealTek螃蟹芯片，是真的扯淡无比。因为自己亲身参与了这个项目，所以知道这个以非标准MIPS架构“闻名”的螃蟹芯片在Openwrt上是如何的不靠谱，尤其是初期引进我司的时候，并且那时还没有其它同事知道Openwrt，更别说了解了，然后项目就这么上马了。。。heheda~~~

虽然参与过这个项目，但是基本工作保密原则，内部代码是不能用的。只能找开源的东西（既然公开了嘛，那就不是我的问题了）。网络上关于螃蟹片子openwrt的资料是少之又少啊，不过幸运的是github上有位哥们上传了最新版本的sdk，比我司拿到的新多了~~~ 看样子是TOTOLINK流出来的？地址[在这里](https://github.com/Wanyor/openwrt-rtk),Openwrt-SDK的直接下载页面[点这里](https://sourceforge.net/projects/rtl819x/files/rtk_openwrtSDK_v2.5.tar.gz/download)

下载完，就解压开始编译了。

<!-- more -->

## 编译SDK
按照SDK中的`UserGuide`所说，初级步骤如下：

### 解压缩sdk后，进入sdk目录
```shell
tar xvzf rtk_openwrtSDK_v2.5.tar.gz
cd rtk_openwrtSDK_v2.5_20160905
tar xvzf rtk_openwrt_sdk.tar.gz
cd rtk_openwrt_sdk
```

### 执行feeds更新命令，更新一下包
```shell
./scripts/feeds update -a
./scripts/feeds install -a
```

### 拷贝预设的配置文件
因为我这准备是给酷开智能路由器编译，所以用的是它的配置方案RTL8197DL。将预设的配置方案导入为将要进行编译的基础配置方案。
```shell
cp rtk_deconfig/defconfig_rtl819xd .config
```

### 加载RTK的PATCH
这里就是扯淡的螃蟹以显示自己与主流架构不同之处了。。。
```shell
./rtk_scripts/rtk_init.sh patch
```

### 详细设置各种配置包
接下来就可以使用命令进入配置界面来选择自己喜欢的各种包了。
```shell
make menuconfig
```

### 编译
配置选择好想要的包后，就可以开始编译了。
```shell
make V=99
```
这里，如果你的电脑或者服务器性能很牛的话，可以加上`-j`参数，来使用CPU的多核心让编译过程更快速。如
```shell
make V=99 -j 64
```
不过建议在第一次编译时尽量不要用编译优化参数。一是因为第一次会要下载很多你选择的包的源文件，另外第一次编译时很多交叉依赖最好让其编译完成一次比较不容易出错。

此外，如果对kernel有什么模块上的需求，也可以单独配置一下kernel。编译过linux kernel的都知道怎么搞。
```shell
make kernel_menuconfig
```

enjoy!
