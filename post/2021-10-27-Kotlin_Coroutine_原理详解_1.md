---
title: Kotlin Coroutine 原理详解（1）--- Coroutine 的诞生背景
categories: Kotlin
---

## 诞生背景

要想了解 `Kotlin Coroutine`，首先需要了解`Coroutine` 这一概念；

### 进程的诞生

上世纪 60 年代早期，计算机控制软件已从监视器控制软件进化为执行控制软件；CPU 跑的越来越快，但是 CPU 并没被很好的利用；此时的矛盾是日益增长的 CPU 性能与落后的多任务处理机制之间的矛盾；

解决此矛盾的方法是设计了分时机制，每个应用程序运行相同的时间片，时间片结束切换到下一个程序，并且保留当前程序运行的上下文；

### 协程的诞生

1958 年，[Melvin Conway](https://en.wikipedia.org/wiki/Melvin_Conway)  在发明了术语 `Coroutine` ，并在一门编译语言中进行实践；随后在 1963 年，在  ["Design of a Separable Transition-diagram Compiler"](http://melconway.com/Home/pdf/compiler.pdf) 中正式阐述了 `Coroutine` 的含义；

大概意思是协程是一个相对独立的函数，当它需要来自其他协程的结果时，就会中断当前协程，发送消息给其他协程，等其他协程计算出结果后，再异步地传回给当前协程；

当前协程在整个过程中有两种状态：

1. 挂起，当前协程中断，CPU 在跑其他协程；
2. 恢复，当前协程拿到结果，继续运行；

在理想情况下，协程操作中不包含系统调用如文件操作，则可以把协程称为 `用户态线程`；

协程是可以使用线程来实现的，但是这样的话，协程切换就会陷入内核态，就失去了`用户态线程`的优势；

### 协程的实现方式

解决挂起恢复的机制；

1. 状态机，挂起后改变当前状态，恢复后根据当前状态执行相关代码；
2. 角色模式，各个角色执行相关的逻辑，但过程的调度由中央调度器负责；
3. 生成器，查看这两篇文档了解 [Generator](https://en.wikipedia.org/wiki/Generator_(computer_programming) ) 、[廖雪峰]([generator - 廖雪峰的官方网站 (liaoxuefeng.com)](https://www.liaoxuefeng.com/wiki/1022910821149312/1023024381818112)) ; 就是一个函数，通过反复调用来获得其在不同阶段的结果；
4. 进程间通信；
5. 反转通信，通常使用在数学软件中；

### Kotlin的协程实现

`Kotlin` 的协程实现是基于线程和状态机的；

```kotlin
fun main() = runBlocking {
    launch {
        val value1 = fun1()
        val value2 = fun2()

        println("time: ${System.currentTimeMillis()} result: ${value1 + value2}")
    }
    
    launch {
        println("time: ${System.currentTimeMillis()} second child coroutine is over")
    }

    println("time: ${System.currentTimeMillis()} block is over")
}

suspend fun fun1(): Int {
    delay(100)
    println("time: ${System.currentTimeMillis()} fun1 ${Thread.currentThread().name}")
    return 1
}

suspend fun fun2(): Int {
    return withContext(Dispatchers.IO) {
        println("time: ${System.currentTimeMillis()} fun2 ${Thread.currentThread().name}")
        2
    }
}
```

运行结果如下：

```
block is over
fun1 main
fun2 DefaultDispatcher-worker-1
result: 3
```

可以看到 `kotlin` 所实现的 `coroutine`具有以下功能：

1. 异步执行子协程（launch 方法大括号里的内容）；
2. 所有的子协程结束后才会结束主协程；
3. 协程中会**依次**调用挂起函数；
4. 协程过程中可以切换**线程**执行挂起函数；

根据上述提供的功能，`kotlin` 所实现的 `coroutine` 需要提供以下基础：

1. 主协程对子协程的控制能力；
2. 在协程过程中挂起恢复的能力；
3. 线程调度的能力；

接下来的一系列文章将会详细讲解 `kotlin` 是如何实现上述能力的。
