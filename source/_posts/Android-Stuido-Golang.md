---
title: Android Studio + Golang
date: 2016-09-08 10:01:44
categories:
 - study
tags:
 - Android
 - Golang
---

简单说下如何在Android上用Golang编译软件。
<!-- more -->

Golang基础版本得高于1.5；Android Studio最好是2.0以上；Java SDK版本最好是1.7以上。
以下配置均以Windows为例。在Windows 10上实践。

## 软件安装
1. 安装Golang。version>1.5。
2. 安装Android Studio。version>2.0。
3. 安装Java SDK。version>1.7。

以上是基础软件安装，到各自的官方网站上下载安装即可。下面要开始配置环境。


## Golang
Golang依赖的最重要的环境变量是`GOPATH`和`GOROOT`。
+ GOROOT：Golang的安装目录。Windows下一般默认为`C:\go`。
+ GOPATH：Golang除`$GOROOT`之外，包含其它项目源码及其二进制文件的目录。

### $GOROOT
#### 示例
添加系统变量GOROOT
![](http://ww1.sinaimg.cn/large/772d7a33gw1f7lzljutz5j20fo00qwel.jpg)

### $GOPATH
`$GOPATH`是`go get`/`go install`等命令的必须设置项。
`$GOPATH`可以是一个目录，也可以是一个目录列表。

`GOPATH`列表中每个目录均包含三个目录src、pkg、bin。
+ `src` 存放源代码（比如*.go *.c *.h *.s等）
+ `pkg` 存放编译的库文件（比如*.a）
+ `bin` 存放可执行文件（为了方便，可以把此目录加入到 $PATH 变量中。如果`$GOPATH`有设置多个目录，那么使用${GOPATH//://bin:}/bin添加所有的bin目录）

`go install`会将可执行文件下载到bin目录。
`go get`命令会将源码下载到src目录，可执行文件下载到bin目录；
> *go系列工具如`get/build/install`等默认会以列表的第一个目录为安装目录。*

#### 示例
添加系统变量GOPATH
![](http://ww4.sinaimg.cn/large/772d7a33gw1f7lzjub1a8j20fo00xt8w.jpg)
如上图所示，我的`$GOPATH`配置了2个目录:
- `C:\Gopath`：用于安装第三方库。如`go get code.google.com/p/go-tour/gotour`会将totour安装到此目录。
- `F:\go`：用于自己写的项目代码。如果在此目录下用`go install`或`go build`来编译安装自己的项目的话，会将项目生成的可执行文件安装到此目录。

**注意：**
*Windows系统中环境变量的形式为%GOPATH%。*
*当go命令搜索包时，它总是第一个从$GOROOT开始。如果找到符合的包名，将不会再去$GOPATH搜索。*
*go get本质上可以理解为首先第一步是通过源码工具clone代码到src下面，然后执行go install。*
**不要将GOPATH设置为与GOROOT同一目录。**

### $PATH
修改环境变量PATH：将%GOROOT%\bin加到环境变量PATH里面，这样就可以直接在dos命令模式下任意目录运行%GOROOT%\bin目录下的程序 如：go.exe godoc.exe。
![](http://ww1.sinaimg.cn/large/772d7a33jw1f7m0nzekzjj20bk030q38.jpg)

## Android Studio
首先下载安装包安装主程序，安装好之后，到安装目录下的tools目录，点击android.bat脚本启动Android SDK Manager工具。由于被墙了，所以要不翻墙，要不找国内代理源，如腾讯。
> 1.启动 Android SDK Manager ，打开主界面，依次选择『Tools』、『Options…』，弹出『Android SDK Manager - Settings』窗口；
> 2.在『Android SDK Manager - Settings』窗口中，在『HTTP Proxy Server』和『HTTP Proxy Port』输入框内填入 android-mirror.bugly.qq.com 镜像服务器地址(不包含http://，如下图)和端口8080，并且选中『Force https://… sources to be fetched using』复选框。设置完成后单击『Close』按钮关闭『Android SDK Manager - Settings』窗口返回到主界面；
> 依次选择『Packages』、『Reload』。
然后你就可以更新最新的tools和buildtools工具了，还需要更新最新的SDK以及支持库文件。

+ 配置Android环境变量`ANDROID_HOME`：
编辑系统环境变量,新建ANDROID_HOME,在变量值一栏中填写你上面的安装路径，比如我的就是`C:\Users\tienchiu\AppData\Local\Android\sdk`;
![](http://ww1.sinaimg.cn/large/772d7a33gw1f7m4prssgnj20ij05adgp.jpg)
+ 配置PATH环境变量：
编辑Path变量，增加`%ANDROID_HOME%\tools`和`%ANDROID_HOME%\platform-tools`;
![](http://ww3.sinaimg.cn/large/772d7a33gw1f7m4pxqxfaj20bx01gwen.jpg)
+ 验证
验证一下你的环境是不是配置好了，启动命令行，输入命令adb version，看是否有信息输出,如果能看到正确的版本信息，那就说你的环境配置没有问题。



## JAVA
java环境有2个参数要配置：
+ `JAVA_HOME ` java安装目录。
+ `CLASSPATH` 

在系统环境变量里添加`JAVA_HOME`变量如下:
![](http://ww1.sinaimg.cn/large/772d7a33gw1f7m118os3rj20fv014aag.jpg)
在系统环境变量里添加`CLASSPATH`变量如下:
![](http://ww1.sinaimg.cn/large/772d7a33gw1f7m11lt9f2j20ij05aab2.jpg)
在系统环境变量`PATH`变量后添加java的可执行目录
![](http://ww2.sinaimg.cn/large/772d7a33gw1f7m11fhwb4j20b000vwee.jpg)

测试一下，任意打开一个dos界面，`java`命令是否可用
![](http://ww3.sinaimg.cn/large/772d7a33gw1f7m16fimjkj20g30ajabf.jpg)
测试一下，任意打开一个dos界面，`javac`命令是否可用
![](http://ww2.sinaimg.cn/large/772d7a33gw1f7m16m1asyj20g10ahjtg.jpg)

**注意：**
**安装完成后记得重启电脑！**


## 参考文献 
[GOPATH与工作空间](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/01.2.md)
[Go语言介绍 - 3：GOPATH](http://qizhanming.com/blog/2012/06/03/go-intro-3-gopath/)
[Golang学习之GOROOT、PATH、GOPATH及go get](http://my.oschina.net/yearnfar/blog/187266)
[Android编译环境配置](http://frank-zhu.github.io/android/2015/10/28/android-build-config/)
