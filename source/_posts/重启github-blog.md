---
title: 重启github blog
date: 2016-07-14 13:44:20
tags:
---
折腾了一上午，赶紧记录下来。

---

打开备份的文件夹，里面存储了原来的部署。换电脑或者重新开始部署时，以下几步是需要的：

1. 安装[NodeJS](https://nodejs.org/en/)。
2. 安装Git。我在windows上用的是[MinGW32](http://www.mingw.org/)。
3. 在文件夹中打开MinGW32窗口，安装npm以及deploye git插件。
```
npm install
npm install hexo-deployer-git --save
```
4. 然后就是正常的写文章，部署了。（最好先把之前的.deploy_git目录删除）
```
hexo clean
hexo new "title"
hexo generate (hexo ge)
hexo deploy (hexo de)
```
5. 发布前可以先本地浏览器预览一下。地址为http://localhost:4000 
```
hexo server (hexo se)
```

PS:有一个很重要的地方要注意，`_config.yml`文件中的deploy地址很模糊，有的文章说是要填`https://github.com/XXX/XXX.github.io.git`，有的文章又说要填`git@github.com:xxxxx/xxxxx.github.io.git`这样的地址，搞不懂为什么。不过我是填的前者。（难道hexo V3之后改了？）
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/cgoder/cgoder.github.io
  branch: master
```
