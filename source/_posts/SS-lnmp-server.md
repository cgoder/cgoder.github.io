---
title: SSpanel @ lnmp server
date: 2017-05-15 09:34:50
categories:
 - work
tags:
 - linux
 - shadowsocks
 - server
---
sspanel v3 魔改版 一些小坑。
<!-- more -->
周末手贱把VPS硬盘给重置了，数据全没了。
只能重新来布署sspanel以及shadowsocks服务了。
正好趁此机会，把sspanel换成魔改版吧。增加的有些功能还是挺有用的，比如验证码。。。

sspanel的布置过程中，有些小坑，顺便记录一下。

1. lnmp安装后，修改nginx的vhost config文件。大部分教程中都提到要在vhost config里加上下面这句伪静态规则，
```bash
location / 
{
	try_files $uri $uri/ /index.php$is_args$args;		                
}
```
其实是不用的，直接vhost config文件里的这句`#include none.conf`改成`#include wordpress.conf`就行了。不但解决了伪静态规则问题，还解决了lnmp架构下，只能访问魔改版主页，子页面都404的问题。
不过，修改root路径还是需要的`root /home/wwwroot/你的域名/public;`，这个`public`不可少！

2. 魔改版前端页面的配置文件，以及后端的SSR的配置文件中，很多数据要统一。
比如`NODE_ID`，以及SS的数据库名，用户名，密码等。
另外，如果前端和后端是在同一个服务器(比如我就是用的同一台VPS)，那连接方式还是SQL方式吧，WEBAPI方式出错概率太大。