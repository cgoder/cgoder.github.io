---
title: Hexo部署时出现"Connection closed by remote host"
date: 2017-03-30 11:52:55
categories:
 - study
tags:
 - hexo
 - github
---
怎么解决SSH连接github被拒绝的问题？
<!-- more -->

- 首先，要确定，网络能正常访问github。
- 其次，要确定，秘钥已经填入github的允许列表。

**如果还连不上github怎么办？**

今天就碰到这事了。
昨天写了篇 **{% post_link Visual-Studio-Code-compile-c-c 使用Visual Studio Code编译c/c++源码 %}** 的文章，能正常`git push`，但是`Hexo d`却一直提示出错，连接不上github。
此时网络是能正常访问github页面的，而且一切github操作都正常，只有hexo部署不正常。
试了下`ssh -vT git@github.com`，结果如下：
```bash
$ ssh -vT github.com
OpenSSH_6.6.1, OpenSSL 1.0.1m 19 Mar 2015
debug1: Connecting to github.com [192.30.253.113] port 22.
debug1: Connection established.
debug1: identity file /c/Users/heheda/.ssh/id_rsa type 1
debug1: identity file /c/Users/heheda/.ssh/id_rsa-cert type -1
debug1: identity file /c/Users/heheda/.ssh/id_dsa type -1
debug1: identity file /c/Users/heheda/.ssh/id_dsa-cert type -1
debug1: identity file /c/Users/heheda/.ssh/id_ecdsa type -1
debug1: identity file /c/Users/heheda/.ssh/id_ecdsa-cert type -1
debug1: identity file /c/Users/heheda/.ssh/id_ed25519 type -1
debug1: identity file /c/Users/heheda/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.6.1
ssh_exchange_identification: Connection closed by remote host
```
出错原因是连接被github中断。
但是很奇怪的是github其它`push/clone`操作都是正常的。那是为什么呢？
google一下，发现[v2ex](https://www.v2ex.com/t/290545)上有人也出现同样的问题，文章中提到了一个对github站点单独配置`config`文件，将github网站的SSH请求从22端口转到443端口去。
好，我就一下看看。先来到主目录下的ssh目录下，创建`config`文件。
```bash
heheda@HEHEDA ~/.ssh
$ vim config
```
在`config`文件里填入如下内容
```bash
Host github.com
  Hostname ssh.github.com
  Port 443
```
保存即可。然后我们再用`ssh -vT git@github.com`试一下效果
```bash
$ ssh -vT git@github.com
OpenSSH_6.6.1, OpenSSL 1.0.1m 19 Mar 2015
debug1: Reading configuration data /c/Users/heheda/.ssh/config
debug1: /c/Users/heheda/.ssh/config line 1: Applying options for github.com
debug1: Hostname has changed; re-reading configuration
debug1: Reading configuration data /c/Users/heheda/.ssh/config
debug1: Connecting to ssh.github.com [192.30.253.122] port 443.
debug1: Connection established.
debug1: identity file /c/Users/heheda/.ssh/id_rsa type 1
debug1: identity file /c/Users/heheda/.ssh/id_rsa-cert type -1
debug1: identity file /c/Users/heheda/.ssh/id_dsa type -1
debug1: identity file /c/Users/heheda/.ssh/id_dsa-cert type -1
debug1: identity file /c/Users/heheda/.ssh/id_ecdsa type -1
debug1: identity file /c/Users/heheda/.ssh/id_ecdsa-cert type -1
debug1: identity file /c/Users/heheda/.ssh/id_ed25519 type -1
debug1: identity file /c/Users/heheda/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.6.1
debug1: Remote protocol version 2.0, remote software version libssh-0.7.0
debug1: no match: libssh-0.7.0
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client aes128-ctr hmac-sha1 none
debug1: kex: client->server aes128-ctr hmac-sha1 none
debug1: sending SSH2_MSG_KEX_ECDH_INIT
debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
debug1: Server host key: RSA 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48
debug1: Host '[ssh.github.com]:443' is known and matches the RSA host key.
debug1: Found key in /c/Users/heheda/.ssh/known_hosts:4
debug1: ssh_rsa_verify: signature correct
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /c/Users/heheda/.ssh/id_rsa
debug1: Server accepts key: pkalg ssh-rsa blen 535
debug1: key_parse_private2: missing begin marker
debug1: read PEM private key done: type RSA
debug1: Authentication succeeded (publickey).
Authenticated to ssh.github.com ([192.30.253.122]:443).
debug1: channel 0: new [client-session]
debug1: Entering interactive session.
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
Hi heheda! You've successfully authenticated, but GitHub does not provide shell access.
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 4032, received 2040 bytes, in 0.7 seconds
Bytes per second: sent 5826.6, received 2948.0
debug1: Exit status 1
```
只要出现`Hi heheda! You've successfully authenticated, but GitHub does not provide shell access.`这句，就表示OK了。

现在又可以愉快地用`hexo d -g`来部署了！