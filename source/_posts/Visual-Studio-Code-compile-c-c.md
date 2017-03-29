---
title: 使用Visual Studio Code编译c/c++源码
date: 2017-03-29 15:51:26
categories:
 - study
tags:
 - compile
---
在Visual Studio Code上如何编译C/C++源码？
<!-- more -->

之前一直在用Visual Studio Code(VSC)来取代Sublime text做Golang的编辑器和调试工具。这次突然想用VSC来编译C/C++的源码，看能不能取代VS的地位，毕竟VS是越来越大了，占用内存又高，速度又慢，堪比AS...

## 下载VSC程序
先安装VSC，官方提供安装包和ZIP包[在此](https://code.visualstudio.com/Download)。
建议直接选ZIP包就可以。

## 安装扩展
打开VSC程序，使用`ctrl+p`直接打开命令面板，输入`ext install c++`命令，即可安装C/C++的扩展支持。建议选择官方Microsoft发行的扩展。
安装完后，代码高亮，代码提示之类的功能都有了。但是还不能直接调试。

## 调试设置`launch.json`
要想调用，得配合`Cygwin/MinGw`一起才行。网上很多安装配置的教程，直接参考就行，例如[这篇](http://blog.csdn.net/c_duoduo/article/details/51615381)。不过过程还是有点弯路的。 
在配置好MinGw后，按F5，选择调试模式，并不会有什么结果，那是因为`launch.json`文件里默认并没有`miDebuggerPath`这一项，需要自己添加进去，并填好gdb的路径，如`C:\\MinGW\\bin\\gdb.exe`。
> 如果你发现没有`launch.json`文件，那是因为你的源码不是在一个目录里。你需要把你的源码放到一个目录里，再在VSC里把这个目录添加进来。这时按F5运行或者调试，才会有`launch.json`文件自动生成。

`launch.json`内容如下：
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "C++ Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceRoot}/a.out",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceRoot}",
            "environment": [],
            "externalConsole": true,
            "preLaunchTask": "g++",
            "linux": {
                "MIMode": "gdb",
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            },
            "osx": {
                "MIMode": "lldb"
            },
            "windows": {
                "MIMode": "gdb",
                "miDebuggerPath": "C:\\MinGW\\bin\\gdb.exe", 
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            }
        },
        {
            "name": "C++ Attach",
            "type": "cppdbg",
            "request": "attach",
            "program": "${workspaceRoot}/a.out",
            "processId": "${command:pickProcess}",
            "linux": {
                "MIMode": "gdb",
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            },
            "osx": {
                "MIMode": "lldb"
            },
            "windows": {
                "MIMode": "gdb",
                "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                ]
            }
        }
    ]
}
```

## 调试设置`task.json`
在设置好`launch.json`后，实际上调试还是不行的。开始调试后会提示你找不到对应的目标文件如`a.out`之类的。
那是因为在默认生成的`launch.json`里也没有`preLaunchTask`这一项(上面贴的其实已经添加好了)，而这一项，是用来控制生成`task.json`文件的，这个文件里描述了编译目标文件的内容。我们如上面贴的文件一样添加好之后，再进行调试就可以正常运行了。
这里自动生成的文件就是`task.json`，就是在调试的时候自动编译生成目标文件，对此目标文件进行调试。
`task.json`内容如下：
```json
{
    "version": "0.1.0",
    "command": "g++",
    "args": ["-g", "${file}", "-o", "main.exe"], // 编译命令参数
    "problemMatcher": {
        "owner": "cpp",
        "fileLocation": ["relative", "${workspaceRoot}"],
        "pattern": {
            "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
            "file": 1,
            "line": 2,
            "column": 3,
            "severity": 4,
            "message": 5
        }
    }
}
```
配置好之后，就可以愉快地进行调试了。enjoy~~~
