---
title: golang init函数及调用顺序
date: 2017-03-22 14:48:52
categories:
 - study
tags:
 - golang
---
golang的init函数调用顺序问题实际测试。
<!-- more -->

为了弄清楚golang的init函数调用顺序问题，实际作了一个简单的测试。

## 测试环境
Ubuntu 14.04 64bit版本，安装golang 1.6。
```bash
root@iZgen2bsbstwf4Z:~/test/got# go version
go version go1.6.2 linux/amd64
```

## 测试条件
创建三个不同的golang源文件，分别为a.go/b.go/c.go。且处于同一package中。
```bash
root@iZgen2bsbstwf4Z:~/test/got# ls
a.go  b.go  c.go
```

三个文件的内容分别如下:
**a.go**
```bash
root@iZgen2bsbstwf4Z:~/test/got# cat a.go
package main

import (
"fmt"
_ "test"
)

func init() {
fmt.Println("printf a init.")
}

func main() {
fmt.Println("printf a main.")
//printB()
//test.print1()
//test.print2()
}

```

**b.go**
```bash
root@iZgen2bsbstwf4Z:~/test/got# cat b.go
package main

import (
"fmt"
_ "test"
)

func init() {
fmt.Println("printf b init.")
}

func printB() {
fmt.Println("printf main b printB.")
}

```


**c.go**
```bash
root@iZgen2bsbstwf4Z:~/test/got# cat c.go
package main

import (
"fmt"
)

func init() {
fmt.Println("printf c init.")
}

func printC() {
fmt.Println("printf PC printC.")
}

func printC1() {
fmt.Println("printf PC printC1.")
}

```

## 测试方法

### 直接go build
```bash
root@iZgen2bsbstwf4Z:~/test/got# go build
root@iZgen2bsbstwf4Z:~/test/got# ls
a.go  b.go  c.go  got
```
直接生成可执行文件`got`。直接运行之，可得结果
```bash
root@iZgen2bsbstwf4Z:~/test/got# ./got
printf test init.
printf a init.
printf b init.
printf c init.
printf a main.
```
init函数的调用顺序是a/b/c，说明相同package内init函数调用顺序*可能*是**按照文件名字母排序**来先后调用的。

### 按a/b/c的顺序build
```bash
root@iZgen2bsbstwf4Z:~/test/got# go build -v a.go b.go c.go
test
command-line-arguments
root@iZgen2bsbstwf4Z:~/test/got# ls
a  a.go  b.go  c.go  got
```
运行刚生成的可执行谁的`a`，可得结果
```bash
root@iZgen2bsbstwf4Z:~/test/got# ./a
printf test init.
printf a init.
printf b init.
printf c init.
printf a main.
```
init函数的调用顺序是a/b/c，证明相同package内init函数调用顺序是**按照文件名字母排序**来先后调用的。

### 按a/c/b的顺序build
```bash
root@iZgen2bsbstwf4Z:~/test/got# go build a.go c.go b.go
root@iZgen2bsbstwf4Z:~/test/got# ./a
printf test init.
printf a init.
printf c init.
printf b init.
printf a main.
```
生成的可执行文件仍然是`a`。运行之，可得结果
init函数的调用顺序是a/c/b，init函数的调用顺序是**按照build文件的先后顺序**来调用的。

### 按b/a/c/的顺序build
```bash
root@iZgen2bsbstwf4Z:~/test/got# go build b.go a.go c.go
root@iZgen2bsbstwf4Z:~/test/got# ./b
printf test init.
printf b init.
printf a init.
printf c init.
printf a main.
```
生成的可执行文件是`b`。运行之，可得结果
init函数的调用顺序是b/a/c，init函数的调用顺序是**按照build文件的先后顺序**来调用的。

### 按b/c/a的顺序build
```bash
root@iZgen2bsbstwf4Z:~/test/got# go build b.go c.go a.go
root@iZgen2bsbstwf4Z:~/test/got# ./b
printf test init.
printf b init.
printf c init.
printf a init.
printf a main.
```
生成的可执行文件仍然是`b`。运行之，可得结果
init函数的调用顺序是b/c/a，init函数的调用顺序是**按照build文件的先后顺序**来调用的。

### 按c/a/b的顺序build
```bash
root@iZgen2bsbstwf4Z:~/test/got# go build c.go a.go b.go
root@iZgen2bsbstwf4Z:~/test/got# ./c
printf test init.
printf c init.
printf a init.
printf b init.
printf a main.
```
生成的可执行文件是`c`。运行之，可得结果
init函数的调用顺序是c/a/b，init函数的调用顺序是**按照build文件的先后顺序**来调用的。

### 按c/b/a的顺序build
```bash
root@iZgen2bsbstwf4Z:~/test/got# go build c.go b.go a.go
root@iZgen2bsbstwf4Z:~/test/got# ./c
printf test init.
printf c init.
printf b init.
printf a init.
printf a main.
```
生成的可执行文件仍然是`c`。运行之，可得结果
init函数的调用顺序是c/a/b，init函数的调用顺序是**按照build文件的先后顺序**来调用的。

## 结论
### 相同package下init函数调用顺序
- 如果没有指定build时的源文件顺序，默认是按照文件名的字母顺序排序来编译的，所以init的调用顺序是按源文件名排序先后来调用。
- 如果指定build时源文件顺序，那么就严格按照源文件编译时排序来先后调用。
~~那么，是不是可以说，相同package下init函数调用顺序取决于源文件的编译顺序呢？~~

### 不同package下init函数调用顺序
- 先调用被import进来的package里的init函数。如图所示
![](http://ww1.sinaimg.cn/large/772d7a33jw1f7m9l74lqvj20qc0bnjta.jpg)
**被import进来的package里不同文件的init函数遵循相同package下init调用顺序。**
- 最后调用本package里的init函数。同时也遵循相同package下init调用顺序。

> 0. init函数是可选的，不是必须实现的。
> 1. init函数是用于程序执行前做包的初始化的函数，比如初始化包里的变量常量等。
> 2. 每个包可以拥有多个init函数。
> 3. 包的每个源文件也可以拥有多个init函数
> 4. 所有的init函数及都在main函数之前被自动调用。不可被其它函数调用。
> 5. ~~相同包的init函数按照源文件编译顺序决定执行顺序。（源文件编译默认是按照文件名排序）~~
> 6. 不同包的init函数按照包导入的依赖关系决定执行顺序。
> 7. 导入包，可以只调用init函数和常量，却不使用package内其它资源。如上述文章import test包。
> 8. 一个包可能被多个包同时导入，但是最终只会被导入一次，init函数同样只初始化一次。如上述文章import fmt包。
