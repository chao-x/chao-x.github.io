---
title: "初探内存泄露"
date: 2022-08-23T12:07:24+08:00
draft: false
tags:
- go
- 内存泄露
categories:
- go
toc: true
---
## 1.定义

 **内存泄露 memory leak** ，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但持续累积，内存打满以后，服务将会崩溃。

*监控实例：*

![](http://rte.weiyun.baidu.com/api/imageDownloadAddress?attachId=60810991bd67426faadc33c895f6c62d)

正常的内存监控曲线应该稳定在一个区间内起伏。而内存泄露后，可以看到，内存一直都在上涨，在打满后，实例直接崩溃重启，然后又重新打满，又重启，如此循环。

## 2.内存泄露的常见原因

1. 全局变量应该释放的内存没有被释放，全局变量的内存一直增长(正常应该有释放有增长)，如map的kv一直在增加、切片的长度一直在增长
2. 需要close的变量没有被关闭，如http 链接、mysql链接
3. goroutine 泄露，原本应该结束的goroutine没有结束，导致对应的内存无法被释放
4. Cgo 相关的内存泄露，在Go里使用Cgo，C代码分配的内存是无法被Go的GC管理的，也无法被pprof追踪

## 3.使用pprof

### pprof介绍

pprof是 Go 语言中分析程序运行性能的工具，它能提供各种性能数据：

![](http://rte.weiyun.baidu.com/api/imageDownloadAddress?attachId=65677d01b08641699f15b6e9022416a5)

### 建议排查步骤

#### 一.检查代码

查看全局变量是否有可疑逻辑，导致全局变量内存不断增长

查看是否有没有关闭的***close***

查看**Cgo** **相关代码**，如CString的内存有没有被释放

---

 **实例：** 函数使用CString匿名参数，也会导致内存泄露

问题代码：

```go
ret := C.get_value(C.CString(file), C.CString(key), &content[0], 40960, &errmsg[0], 1024)
```

修复后：

```go
cfile := C.CString(file)
ckey := C.CString(key)
ret := C.get_value(cfile, ckey, &content[0], 40960, &errmsg[0], 1024)
C.free(unsafe.Pointer(cfile))
C.free(unsafe.Pointer(ckey))
```

---

等等其他可能造成内存泄露的代码

#### 二.查看goroutine是否泄露

压测过程中，使用 ***pprof*** ，访问 （对应的IP和端口）[http://127.0.0.1:8305/debug/pprof/goroutine?debug=1](http://10.12.205.56:8305/debug/pprof/goroutine?debug=1)

查看goroutine数量是否稳定在一个区间：

如果goroutine不断增长，确定为***goroutine*****泄露**

#### 三.查看实时内存

压测过程中，当内存达到一个高值以后，使用命令 （对应IP和端口）

生成实时内存数据：

```shell
go tool pprof http://127.0.0.1:8080/debug/pprof/heap?debug=1
```

安装并使用graphviz 绘制对应的图形

使用命令：

```shell
go tool pprof http://127.0.0.1:8080/debug/pprof/heap?debug=1
```

访问 [http://127.0.0.1:8090/ui](http://127.0.0.1:8090/ui) 查看内存路线图和火焰图，帮助定位问题

火焰图的查看方法为：从上到下为调用链路，左右间距越大标识占用内存越多

#### 四.查看累积内存

如果实时内存无法定位问题，可以查看累积时间申请的内存，帮助定位问题

```bash
go tool pprof http://127.0.0.1:8080/debug/pprof/heap -seconds 30
```

## 4.内存监控

没有对应的内存监控工具，可以使用命令

```bash
ps aux | grep 对应pid
```

查看**RSS**查看实时占用内存

## 5.查看GC

查看GC是否正常，使用命令

```bash
GODEBUG=gctrace=1 go run main.go
```
