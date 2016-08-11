---
title: IPSEC/IKEV2 vpn fo vps
date: 2016-08-11 15:49:44
categories:
 - study
tags:
 - vpn
 - vps
---
前阵子在VPS上搭了个IPSEC的vpn服务器，主要用于没越狱的iOS8上。简单记录一下过程。
主要有牛人已经做好脚本了，我几乎只需要照着步骤，一路按回车就行。
前人的blog在[这儿](https://quericy.me/blog/699/)，Github的脚本页面在[这儿](https://github.com/quericy/one-key-ikev2-vpn)。

SSH连接到VPS上，直接wget下来脚本，修改下权限，执行脚本，按着提示一路回车就OK。
> wget --no-check-certificate https://raw.githubusercontent.com/quericy/one-key-ikev2-vpn/master/one-key-ikev2.sh

```shell
chmod +x one-key-ikev2.sh
./one-key-ikev2.sh
```

**注意项：**
1. 在执行脚本的时候，要注意，选对自己VPS的架构，不然就哦哦了。
2. 安装完，记得把启动命令`/usr/local/sbin/ipsec start`加入到系统启动脚本(/etc/rc.local)里。
3. 安装完，如果不想用默认的密码，可以修改`/usr/local/etc/ipsec.secrets`文件中的密码。形如
```shell
root@localhost:~# cat /usr/local/etc/ipsec.secrets
: RSA server.pem
: PSK "IPSECXXXX"
: XAUTH "IPSECXXXX"
guest : EAP "pwd4guest"
myboss : EAP "bigboss"
```
4. 如果不需要了，卸载要在脚本下载下来并解压的`strongswan-5.3.5`目录里直接`make uninstall`即可。
