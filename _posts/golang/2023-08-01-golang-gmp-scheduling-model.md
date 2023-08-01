---
layout:       post
title:        "并发的艺术：Go语言中的GMP调度模型解析"
subtitle:     "How will property taxes change our lives?"
header-img:   "img/in-post/head/2023-08-01-golang-gmp-scheduling-model.jpeg"
date:         2023-08-01 00:00:00
author:       "Xic"
tags:
    - Golang
    - GMP
    - 底层原理
---
# 引言

在这篇文章中，我们将探讨Go语言中的并发编程模型 —— GMP调度模型。我们将深入研究其内部构成、工作流程，并探讨其优势和劣势。

# Go语言与并发编程

Go语言是一门强大的编程语言，以其出色的并发编程能力而闻名。Go语言的并发是通过Goroutine实现的，Goroutine是一种轻量级线程，创建和切换的成本低，使得Go语言在处理并发任务时更加高效。

## Go语言简介

Go语言，也称为Golang，是由Google开发的一门静态类型，编译型语言。它的语法简洁，易于学习，同时它对并发编程的支持使它在服务端开发中得到了广泛的应用。

## Go的并发编程特性

Go的并发是通过Goroutine和Channel来实现的。Goroutine类似于线程，但它消耗的资源少得多。Channel则提供了一种安全地在Goroutine之间共享数据的方式。

## Goroutine


每一个Go程序至少有一个Goroutine：主Goroutine。当程序启动时，主Goroutine就会自动创建。新的Goroutine可以通过go关键字来创建。Goroutine的执行不会阻塞其它Goroutine。下面的代码就是一个简单的Goroutine实例：

```go
package main

import (
    "fmt"
    "sync"
)

func hello(wg *sync.WaitGroup, i int) {
    defer wg.Done()
    fmt.Println("Hello from Goroutine", i)
}

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1) //添加一个Goroutine
        go hello(&wg, i) //创建并启动Goroutine
    }
    wg.Wait() //等待所有Goroutine执行完毕
}
```

在这个示例中，我们在 `main` 函数中创建了5个Goroutine。每个Goroutine都会执行hello函数并打印一条消息。我们使用 `sync.WaitGroup` 来确保所有的Goroutine都执行完毕。

## 理解GMP调度模型的构成

GMP模型由三部分构成：Goroutine(G)，系统线程(M)，调度器(P)。

> **Goroutine(G)**：在Go语言中，每个并发的任务都被称为一个Goroutine。Goroutine是用户级线程，而不是系统线程，所以它的创建、销毁和切换的成本低于系统线程。   
> **系统线程(M)**： 是内核级线程，每个系统线程都对应一个内核级线程。  
> **调度器(P)**：负责调度，它保存了一组Goroutine。P的数量默认等于CPU的数量。  

![GMP模型构成](/img/in-post/article-pic/gmp_storage_structure.png)

# GMP调度模型的详细工作流程

在GMP模型中，M执行G需要一个P的上下文环境。每个P都有一个本地的Goroutine队列，M需要从P关联的G队列中获取G去执行，当M关联的P没有G可运行时，M会尝试从其他P的队列中偷取G来运行。

## 创建Goroutine

当新的Goroutine被创建时，会先将其放在当前P的本地队列中，如果P的本地队列已满，那么会把新的Goroutine放到全局队列中。

## 调度执行

M从其关联的P的队列中获取G执行，如果P的队列为空，则尝试从全局队列或者其它P的队列中偷取G执行。

## G的阻塞与唤醒

如果G因为系统调用或者等待资源而阻塞，那么M会将G置为等待状态，然后去P的队列中获取新的G执行。当阻塞的G被唤醒时，会将G放入全局队列或者其它P的队列中，然后唤醒或创建一个M去执行这个G。

# GMP调度模型的优势与劣势

## 优势

1. 高并发：Go语言使用Goroutine取代了传统的线程模型，可以支持数以百万计的Goroutine并发执行。
2. 高性能：由于Goroutine的设计和GMP调度模型的优化，Go语言在CPU和内存的利用率上有很高的效率。

## 劣势

尽管 Go 的 GMP 模型在很多情况下表现优秀，但在某些情况下，比如CPU密集型任务，Go的并发可能不会有明显的性能优势。

# 总结

GMP调度模型是Go语言并发编程的基础。通过深入理解GMP模型的工作原理，我们可以更好地利用Go的并发特性，编写出更高效的并发程序。

