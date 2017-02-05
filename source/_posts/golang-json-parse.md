---
title: golang json parse
date: 2016-09-21 16:51:25
categories:
 - study
tags:
 - Json
 - Golang
---

golang怎么解析json?

<!-- more -->
golang下的json分析很方便，有内置的包`encoding/json`可以使用。但是在使用中要注意json的分析一些小细节。比如json对象的名字要与实际名字一样（当然也可以结构体名不一样，而用标签`TAG`来保证名字一样），并且**结构体的成员变量名首字母要大写**。至于为什么？别忘了golang的规则是`首字母大写的变量代表能够被导出`。

例如:
```go
type User struct {
    Name      string
    PostCount int    `json:"posts-total"`
    PostList  []Post `json:"posts"`
}

type Post struct {
    ID        json.Number `json:"id"`
    URL       string      `json:"url"`
    Type      string      `json:"type"`
    Slug      string      `json:"slug"`
    Timestamp int64       `json:"unix-timestamp"`
}

var contents = `{"post-ver":1,"posts-total":7,"posts":[{"id":"148619653632","url":"http:\/\/test.tumblr.com\/post\/148619653632"}]};`

```
这里面`User/Post`要解析的成员变量必须大写字母开头，否则就不能解析json到结构体变量内。
另外，这里使用了golang的struct的标签语法`json:"posts-total"`，像这样就可以把原始json里的`posts-total`数据直接存储到`User.PostCount`里。

示例代码参见[这里](https://github.com/cgoder/Gtang)
