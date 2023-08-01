---
layout:       post
title:        "并发的艺术：Go语言中的GMP调度模型解析"
subtitle:     "The Art of Concurrency: Analysis of GMP Scheduling Model in Golang"
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

Go语言的并发编程特性主要表现在Goroutine和Channel两个方面。这两者是Go语言设计者为解决并发问题所特别设计的工具，它们的设计理念源于CSP（Communicating Sequential Processes）并发模型。

1. **Goroutine**

   Goroutine可以看作是一种轻量级的线程，它们在Go语言的运行时环境中被调度和执行。相较于传统的线程，Goroutine的创建和销毁的开销更小，更易于管理。每个Go程序至少有一个Goroutine（主Goroutine）在运行。通过并发的Goroutines，Go程序可以同时处理多个任务，从而提高程序的性能和响应能力。

2. **Channel**

   Channel是Go语言提供的一种数据结构，它提供了一种强大的、安全的在Goroutines之间通信和同步的方式。使用Channel，我们可以在一个Goroutine中发送数据，而在另一个Goroutine中接收数据，这样可以实现Goroutines之间的数据共享。通过Channel的发送和接收操作，Goroutines之间还可以进行同步。这种"**以通信来共享内存**"的思想，有别于传统的"通过共享内存来通信"的模式，让并发程序的设计更加简洁和安全。

Go语言的并发特性允许开发者更简单，更直观地编写并发程序。相较于其他语言，使用Goroutine和Channel编写并发程序不仅更为直观和简洁，也更安全，更容易避免并发程序中常见的问题，如竞争条件、死锁等。这使得Go语言在需要大规模并发的场景下，如网络编程、云计算等，表现出极高的生产力。

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

![GMP模型构成](/img/in-post/article-pic/gmp_storage_structure.png)

GMP模型由三部分构成：Goroutine(G)，系统线程(M)，调度器(P)。

> **Goroutine(G)**：在Go语言中，每个并发的任务都被称为一个Goroutine。Goroutine是用户级线程，而不是系统线程，所以它的创建、销毁和切换的成本低于系统线程。   
> **系统线程(M)**： 是内核级线程，每个系统线程都对应一个内核级线程。  
> **调度器(P)**：负责调度，它保存了一组Goroutine。P的数量默认等于CPU的数量。

# GMP调度模型的详细工作流程

![GMP调度流程](/img/in-post/article-pic/gmp_scheduling_process.png)

1. **Goroutine创建**

   当一个新的Goroutine被创建时，它首先会被放入P的本地队列中。如果本地队列已满或P的数量小于 `GOMAXPROCS`，新的Goroutine会被放入全局队列中。如果此时有空闲的M，那么运行时系统会唤醒或创建一个M来执行这个新的Goroutine。

2. **Goroutine执行**

   每个M会从与其关联的P的本地队列中取出一个Goroutine来执行。如果本地队列为空，M会尝试从全局队列或者其它P的队列中偷取Goroutine。

3. **Goroutine阻塞**

   如果一个Goroutine在执行过程中进行了阻塞操作，如I/O操作或系统调用，这个Goroutine会被移到与这个操作相关的阻塞列表中，与此同时，M和P会解除关联，M会尝试去执行其它的Goroutine。

4. **Goroutine唤醒**

   当阻塞的Goroutine被唤醒时，如I/O操作完成或系统调用返回，它会被放入全局队列或者P的本地队列中。如果有空闲的M，运行时系统会唤醒或创建一个M来执行这个Goroutine。否则，它将在队列中等待，直到有M可用。

5. **线程的管理**

   Go运行时系统会动态地创建和销毁M，以适应程序的需要。同时，为了防止创建过多的线程，系统会限制M的最大数量。当M的数量达到上限，新的Goroutine会被放入队列中等待。当有M空闲出来，系统会从队列中取出一个Goroutine来执行。

通过以上的描述，我们可以看出GMP模型是如何有效地管理和调度Goroutine的。这个模型通过将并发和并行结合起来，充分利用了多核CPU的性能，同时也提供了高效的并发编程模型。
# GMP调度模型的优势与劣势

## 优势

1. 高并发：Go语言使用Goroutine取代了传统的线程模型，可以支持数以百万计的Goroutine并发执行。
2. 高性能：由于Goroutine的设计和GMP调度模型的优化，Go语言在CPU和内存的利用率上有很高的效率。

## 劣势

尽管 Go 的 GMP 模型在很多情况下表现优秀，但在某些情况下，比如CPU密集型任务，Go的并发可能不会有明显的性能优势。

# 总结

GMP调度模型是Go语言并发编程的基础。通过深入理解GMP模型的工作原理，我们可以更好地利用Go的并发特性，编写出更高效的并发程序。

# 参考资料
[Scheduling In Go:Part II - Go Scheduler](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part2.html)