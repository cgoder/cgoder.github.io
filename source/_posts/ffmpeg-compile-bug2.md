---
title: ffmpg cheating native code基本类型与封装类
date: 2017-05-11 09:48:21
categories:
 - work
tags:
 - android
 - ndk
 - ffmpeg
---

下面这个API，有什么坑爹之处？
<!-- more -->
```bash
    public native int ffmpeg_clip_do(Double startTime, Double endTime,  String inFile, String outFile);
```

坑大了，自己挖的，哭着填好。

不小心把`double`写成`Double了`，导致native code debug时，一直出现飞栈的情况。
看了半天才看到这个首字母大小写的区别。突然想到首字母大写的都是类，这里不应该传类啊~~~
google一下
> 一个是基本类型，一个是封装类。封装类可以有方法。

我要的只是基本类型而已！这坑踩的。。。
