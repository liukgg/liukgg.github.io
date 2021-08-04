---
title: Godoc In China
date: 2021-08-01 09:00:00
tags: Golang
categories: Golang编程语言
---

如何在国内查看Golang文档
------------------------
如何在国内查看Golang文档，一直是困扰已久的问题，通过摸索汇总了以下几种方法：

### 【推荐】方法一：在本地启动godoc服务

本地执行以下命令，然后就可以在浏览器访问： http://localhost:8888.

``` golang
godoc -http=:8888
```

### 方法二：还是用godoc，但是本地直接利用 godoc 命令
- 本地可以通过 godoc 命令直接来查看相关文档。
```
# 查看 godoc 用法
$ godoc

# 查看某个package用法
$ godoc pkg_name # eg: godoc math
# 以上会直接输出到终端，如果想要写入到文件里更方便查看，也很简单
$ godoc math > file_godoc_math

# 查看某个package的某个函数的用法
$ godoc math Abs
func Abs(x float64) float64
    Abs returns the absolute value of x.

    Special cases are:

	Abs(±Inf) = +Inf
	Abs(NaN) = NaN
```

### 方法三：下载godoc到本地
https://github.com/astaxie/godoc

### 方法四：“科学上网”，然后访问：https://golang.org/doc/

### 文章推荐：Go语言究竟好在哪里？
https://www.infoq.cn/article/jqrMtm15lmCP_lNCJPk3
