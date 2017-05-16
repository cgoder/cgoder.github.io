---
title: ffmpeg @ windows10 linux subsystem
date: 2017-05-15 09:49:40
categories:
 - work
tags:
 - linux
 - ffmpeg
 - windows
---
在Windows 10下利用Windows Subsystem for Linux来编译ffmpeg。
<!-- more -->

## Windows Subsystem for Linux
巨硬最近几年在搞事啊！有Windows Subsystem for Linux，又有Docker on Windows，这是要要搞统一啊！

那什么是WSL？以下来自维基百科的解释：
> Windows Subsystem for Linux（简称WSL）是一个为在Windows 10上能够原生运行Linux二进制可执行文件（ELF格式）的兼容层。它是由微软与Canonical公司合作开发，目标是使纯正的Ubuntu 14.04 "Trusty Tahr"映像能下载和解压到用户的本地计算机，并且映像内的工具和实用工具能在此子系统上原生运行。
WSL提供了一个微软开发的Linux兼容内核接口（不包含Linux代码），来自Ubuntu的用户模式二进制文件在其上运行。
该子系统不能运行所有Linux软件，例如那些图形用户界面，以及那些需要未实现的Linux内核服务的软件。不过，这可以用在外部X服务器上运行的图形X窗口系统缓解。
此子系统起源于命运多舛的Astoria项目，其目的是允许Android应用运行在Windows 10 Mobile上。此功能组件从Windows 10 Insider Preview build 14316开始可用。

简单的说，就是在Windows系统下虚拟一个子Linux系统。目前是Ubuntu系统，版本可升级。可以在上面作任何与Linux有关的任何事情，就像安装了一个虚拟机一样，可以在cmd/powershell里运行Linux原生可执行文件。

## 安装
怎么安装？So easy...
- 把Windows系统升级到Windows 10 1073及以上版本。
- 打开`cmd`或者`powershell`。
- 命令行里输入`bash`按提示下载文件，跟着提示安装，完毕后即可使用。
> Tips 1: 在资源管理器地址栏可直接输入`cmd`或者`powershell`命令，快捷打开命令行界面。
> Tips 2: 在资源管理器地址栏可直接输入`bash`直接打开命令行界面并开始下载安装。
> Tips 3: 如果发现下载速度太慢，打开ss小飞机，开全局代理，墙外速度更快。

## 使用
安装好之后，打开`cmd`或者`powershell`或者`bash`都可以直接使用。
看了下，Ubutun的版本已经更新到16.04.2 LTS了。
那我们来试一下使用感觉怎么样吧。

### 编译ffmpeg
- 先从[ffmpeg](http://ffmpeg.org/)官网下载源码，下载地址在[这儿](http://ffmpeg.org/releases/ffmpeg-3.3.1.tar.bz2);
- ffmpeg编译准备用来给Android用的，所以需要先下载[Android NDK](https://developer.android.com/ndk/downloads/index.html?hl=zh-cn)，下载linux x86 64bit版本的就行了。
- 编写编译脚本文件。进入ffmpeg源码目录，编写`ndk_build_so.sh`文件。内容如下：(路径要根据自己的PC环境改写)
```bash
#!/bin/bash
# 这个脚本文件是直接将ffmpeg编译成单个so库使用。原理是，将ffmpeg编译成多个模块静态库，再通过解包库，再组合打包库，生成一个动态库。方便Android Studio集成使用。
# ffmpeg编译的模块和编译选项，根据自己项目需求来增减。
NDK=/mnt/d/tools/android-ndk-r14b
PLATFORMROOT=$NDK/platforms/android-15/arch-arm/
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64
function build_one
{
./configure \
 --prefix=$PREFIX \
 --enable-static \
 --disable-shared \
 --disable-doc \
 --disable-ffmpeg \
 --disable-ffplay \
 --disable-ffprobe \
 --disable-ffserver \
 --disable-avdevice \
 --disable-doc \
 --disable-symver \
 --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
 --target-os=linux \
 --arch=arm \
 --cc=$TOOLCHAIN/bin/arm-linux-androideabi-gcc \
 --nm=$TOOLCHAIN/bin/arm-linux-androideabi-nm \
 --enable-cross-compile \
 --sysroot=$PLATFORMROOT \
 --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
 --extra-ldflags="$ADDI_LDFLAGS" \
 $ADDITIONAL_CONFIGURE_FLAG
make clean
make -j4
make install
$TOOLCHAIN/bin/arm-linux-androideabi-ld \
    -rpath-link=$PLATFORMROOT/usr/lib \
    -L$PLATFORMROOT/usr/lib \
    -L$PREFIX/lib \
    -soname libffmpeg.so -shared -nostdlib -Bsymbolic --whole-archive --no-undefined -o \
    $PREFIX/libffmpeg.so \
    $PREFIX/lib/libavcodec.a $PREFIX/lib/libavfilter.a \
    $PREFIX/lib/libavformat.a $PREFIX/lib/libavutil.a \
    $PREFIX/lib/libswresample.a $PREFIX/lib/libswscale.a \
    -lc -lm -lz -ldl -llog --dynamic-linker=/system/bin/linker \
    $TOOLCHAIN/lib/gcc/arm-linux-androideabi/4.9.x/libgcc.a  
}
CPU=arm
ADDI_CFLAGS="-marm"
PREFIX=$(pwd)/android/$CPU
build_one
```
- 直接运行`ndk_build_so.sh`文件，即可在ffmpeg源码根目录下生成`android`目录，里面对应不同编译平台生成的库以及头文件。
