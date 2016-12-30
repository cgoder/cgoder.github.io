---
title: binder note
date: 2016-12-30 16:23:52
categories:
 - study
tags:
 - Android
---
#binder note
---

##[通信](http://light3moon.com/2015/01/28/Android%20Binder%20%E5%88%86%E6%9E%90%E2%80%94%E2%80%94%E9%80%9A%E4%BF%A1%E6%A8%A1%E5%9E%8B/)
1. service 运行，阻塞于 ioctl，等待 client 发起请求
2. client 通过 ioctl 发起 IPC 请求，等待 service 结果
2.1. client send `BC_TRANSACTION` —> kernel
2.2. kernel return `BR_TRANSACTION_COMPLETE` —> client
2.3. client 阻塞于 ioctl，等待 service 返回结果
3. service 被唤醒，完成业务，返回结果
3.1. kernel return `BR_TRANSACTION` —> service
3.2. service impl IPC call
3.3. service send `BC_REPLY` —> kernel
3.4. kernel return `BR_TRANSACTION_COMPLETE` —> service
4. client 被唤醒，读取 service 返回结果， IPC 结束
4.1. kernel return `BR_REPLY` —> client
4.2. IPC call end

> BC 开头的协议都是用户空间对 kernel 发送的， BR 开头的协议都是 kernel 返回给用户空间的。所以应用程序是通过 kernel 的 binder 驱动进行通信的（ client 发往 service其实不知道对方彼此的存在）。
向 kernel 发送传送请求的命令`BC_TRANSACTION/BC_REPLY`，kernel 会返回 `BR_TRANSACTION_COMPLETE` 告诉发送者，传送完成。
![](http://ww4.sinaimg.cn/large/772d7a33gw1fb8wancgqbj20m80d3di0.jpg)

