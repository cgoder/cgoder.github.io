---
title: '[hexo deploy error]github key'
date: 2017-05-16 09:13:16
categories:
 - study
tags:
 - hexo
 - github
---
hexo deploy出错，提示git pubkey无效。
<!-- more -->
提示内容如下：
```bash
nothing to commit, working tree clean
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (D:\code\github\cgoder.github.io\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:194:7)
    at ChildProcess.cp.emit (D:\code\github\cgoder.github.io\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:899:16)
    at Socket.<anonymous> (internal/child_process.js:342:11)
    at emitOne (events.js:96:13)
    at Socket.emit (events.js:191:7)
    at Pipe._handle.close [as _onclose] (net.js:511:12)
FATAL Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

Error: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.

    at ChildProcess.<anonymous> (D:\code\github\cgoder.github.io\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:194:7)
    at ChildProcess.cp.emit (D:\code\github\cgoder.github.io\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:899:16)
    at Socket.<anonymous> (internal/child_process.js:342:11)
    at emitOne (events.js:96:13)
    at Socket.emit (events.js:191:7)
    at Pipe._handle.close [as _onclose] (net.js:511:12)
```

挠了好久的头，才发现，原来自己作死，在`ssh-keygen`生成公钥时，给key取了一个另外的名字，不是默认的`rsd.pub`，导致不能识别这个key。重新生成，使用默认名字就行了。
或者用`ssh-add`加上也行。