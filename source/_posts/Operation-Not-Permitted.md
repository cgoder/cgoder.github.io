---
title: Linux删除文件出现"Operation Not Permitted"提示
date: 2017-01-19 14:00:44
categories:
 - study
tags:
 - Linux
---
刚才在aliyun空间上删除[LNMP](https://lnmp.org/)生成的某个文件时，出现某个隐藏文件删除不了，提示"Operation Not Permitted"的问题。但是我是root用户登录的啊！

突然想起来以前见过一篇文章，谈到过linux下面的file flag问题。为了防止某个重要的文件被删除，可以用`chattr`命令将文章的flag改变。当文件flag被改后，即使root用户想要修改或者删除都会提示"Operation not permitted"，这是一种很好的文件保护机制。`chattr`命令详解如下：
> chattr命令的用法：
chattr [ -RVf ] [ -v version ] [ mode ] files…
最关键的是在[mode]部分，[mode]部分是由+-=和[ASacDdIijsTtu]这些字符组合的，这部分是用来控制文件的
属性。
+ ：在原有参数设定基础上，追加参数。
- ：在原有参数设定基础上，移除参数。
= ：更新为指定参数设定。
A：文件或目录的 atime (access time)不可被修改(modified), 可以有效预防例如手提电脑磁盘I/O错误的发生。
S：硬盘I/O同步选项，功能类似sync。
a：即append，设定该参数后，只能向文件中添加数据，而不能删除，多用于服务器日志文件安全，只有root才能设定这个属性。
c：即compresse，设定文件是否经压缩后再存储。读取时需要经过自动解压操作。
d：即no dump，设定文件不能成为dump程序的备份目标。
i：设定文件不能被删除、改名、设定链接关系，同时不能写入或新增内容。i参数对于文件 系统的安全设置有很大帮助。
j：即journal，设定此参数使得当通过mount参数：data=ordered 或者 data=writeback 挂 载的文件系统，文件在写入时会先被记录(在journal中)。如果filesystem被设定参数为 data=journal，则该参数自动失效。
s：保密性地删除文件或目录，即硬盘空间被全部收回。
u：与s相反，当设定为u时，数据内容其实还存在磁盘中，可以用于undeletion。
各参数选项中常用到的是a和i。a选项强制只可添加不可删除，多用于日志系统的安全设定。而i是更为严格的安全设定，只有superuser (root) 或具有CAP_LINUX_IMMUTABLE处理能力（标识）的进程能够施加该选项。

那怎么来使用呢？
```shell
root@root:/home# rm .user.ini 
rm: cannot remove '.user.ini': Operation not permitted
root@root:/home# lsattr .user.ini 
----i--------e-- .user.ini
root@root:/home# chattr .user.ini 
root@root:/home# lsattr .user.ini
-------------e-- .user.ini
root@root:/home# rm .user.ini
```
