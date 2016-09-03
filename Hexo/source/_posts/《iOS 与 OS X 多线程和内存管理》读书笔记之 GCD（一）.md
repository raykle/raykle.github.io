---
title: 《iOS 与 OS X 多线程和内存管理》读书笔记之 GCD（一）
date: 2016-08-16 11:43:41
tags: [iOS, GCD]
categories: 
    - "iOS"
    - "读书笔记"
description: "该篇内容是在阅读《iOS 与 OS X 多线程和内存管理》时做的笔记，加入了一些自己的理解说明和测试 demo，方便查阅。"
---


{% cq %}
书籍链接：[Objective-C 高级编程: iOS 与 OS X 多线程和内存管理](https://www.amazon.cn/Objective-C%E9%AB%98%E7%BA%A7%E7%BC%96%E7%A8%8B-iOS%E4%B8%8EOS-X%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86-%E5%9D%82%E6%9C%AC%E4%B8%80%E6%A0%91/dp/B00DE60G3S/ref=sr_1_1?ie=UTF8&qid=1471501826&sr=8-1&keywords=iOS+%E4%B8%8E+OS+X+%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%92%8C%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86)
{% endcq %}


## 几个概念

`异步`：提交任务之后立刻返回，在后台队列中执行任务。
`同步`：提交的任务执行完之后，再返回。

`并行执行`：提交到任务到队列中之后，比如按顺序提交了任务 1 和任务 2，在任务 1 开始执行之后，不管任务 1 有没有执行完毕，都开始执行任务 2。
`串行执行`：提交到任务到队列中之后，比如按顺序提交了任务 1 和任务 2，只有在任务 1 执行完毕之后，才执行任务 2。

> 异步 or 同步，由函数 `dispatch_sync（同步）` or `dispatch_async（异步）` 决定。
> 并行 or 串行，由队列本身决定。如添加任务到 `Serial Dispatch Queue（串行队列）` or `Concurrent Dispatch Queue（并行队列）` 队列中。

---

## Dispatch Queue

> 开发者要做的只是定义想执行的任务并追加到适当的 Dispatch Queue 中。

```objc
dispatch_async(queue, ^{
    //想要执行的任务
});
```

该源代码使用 Block 语法“定义想执行的任务”，通过 dispatch_async 函数“追加”赋值在变量 queue 的“Dispatch Queue 中”。仅这样就可使指定的 Block 在另一线程中执行。

“Dispatch Queue”是执行处理的等待队列。应用程序编写人员通过 dispatch_async 函数等 API，在 Block 语法中记述想执行的处理并将其追加到 Dispatch Queue 中。Dispatch Queue 按照追加的顺序（**`先进先出 FIFO`**）执行处理。

---


## Dispatch Queue 种类

`Serial Dispatch Queue` 等待现在执行中处理结果，使用一个线程。  
`Concurrent Dispatch Queue` 不等待现在执行中处理结果，使用多个线程

```objc
//创建一个 Serial Dispatch Queue
dispatch_queue_t serialQueue = dispatch_queue_create("come.iBinaryOrg.serial", DISPATCH_QUEUE_SERIAL);

dispatch_async(serialQueue, ^{
    NSLog(@"block 0");
});

dispatch_async(serialQueue, ^{
    NSLog(@"block 1");
});

dispatch_async(serialQueue, ^{
    NSLog(@"block 2");
});

dispatch_async(serialQueue, ^{
    NSLog(@"block 3");
});

...

```

当变量为 Serial Dispatch Queue 时，因为要等待现在执行中的处理结束，所以首先执行 blk0，blk0 执行结束后，接着执行 blk1，blk1 结束后再开始执行 blk2，如此重复。同时执行的处理数只能有 1 个。即执行该源码后，一定按照以下顺序进行处理：

```
block 0
block 1
block 2
block 3
...
```

当变量 queue 为 Concurrent Dispatch Queue 时，因为不用等待现在执行中的处理结果结束，所以首先执行 blk0，不管 blk0 的执行是否结束，都开始执行后面的 blk1，不管 blk1 的执行是否结束，都开始执行后面的 blk2，如此重复循环。

这样虽然不用等待处理结果，可以并行执行多个处理，但并行执行的处理数量取决于`当前系统的状态`。即 iOS 和 OS X 基于 Dispatch Queue 中的处理数、CPU 核数以及 CPU 负荷等当前系统的状态来决定 Concurrent Dispatch Queue 中并行执行的处理数。所谓“并行执行”，就是使用多个线程同时执行多个处理。


---

## dispatch_queue_create

关于 Serial Dispatch Queue 生成个数的注意事项：

- 使用 dispatch_queue_create 函数可生成任意多个 Dispatch Queue，当生成多个 Serial Dispatch Queue 时，各个 Serial Dispatch Queue 将 ***并行执行***。虽然在一个 Serial Dispatch Queue 中同时只能执行一个追加处理，但如果将处理分别追加到4个 Serial Dispatch Queue 中，各个 Serial Dispatch Queue 执行一个，即为 ***同时执行4个处理***。
- 一旦生成 Serial Dispatch Queue 并追加处理，系统对于一个 Serial Dispatch Queue 就只生成并使用一个线程。如果生成 2000 个 Serial Dispatch Queue，那么就生成 2000 个线程。*过多使用多线程，就会消耗大量内存，引起大量的上下文切换，大幅度降低系统的响应性能。*

使用方法：

```objc
//Serial Dispatch Queue
dispatch_queue_t mySerialDispatchQueue = dispatch_queue_create("com.iBinaryOrg.gcd.MySerialDispatchQueue", NULL/* 等同于 DISPATCH_QUEUE_SERIAL，串行队列 */)；

// 异步执行
dispatch_async(mySerialDispatchQueue, ^{
    NSLog(@"Blk on mySerialDispatchQueue");
});

//Concurrent Dispatch Queue
dispatch_queue_t myConcurrentDispatchQueue = dispatch_queue_create("com.iBinaryOrg.gcd.MyConcurrentDispatchQueue", DISPATCH_QUEUE_CONCURRENT)；// 并行队列

// 异步执行
dispatch_async(myConcurrentDispatchQueue, ^{
    NSLog(@"Blk on myConcurrentDispatchQueue");
});

```

---

## Main Dispatch Queue / Global Dispatch Queue

> Main Dispatch Queue 是 Serial Dispatch Queue。
> Global Dispatch Queue 是 Concurrent Dispatch Queue。  


**系统提供的 Dispatch Queue 种类：**

名称 | Dispatch Queue 种类 | 说明
---|---|---
Main Dispatch Queue | Serial DIspatch Queue | 主线程执行
Serial Dispatch Queue (Hign Priority) | Concurrent Dispatch Queue | 执行优先级：高 (最高优先)
Serial Dispatch Queue (Default Priority) | Concurrent Dispatch Queue | 执行优先级：默认
Serial Dispatch Queue (Low Priority) | Concurrent Dispatch Queue | 执行优先级：低
Serial Dispatch Queue (Background Priority) | Concurrent Dispatch Queue | 执行优先级：后台

**使用举例**

```objc
//在默认优先级的 Global Dispatch Queue 中执行block
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    /*
     *可并行执行的处理
     */
    
    /*
     *在 Main Dispatch Queue 中执行 block
     */
    dispatch_async(dispatch_get_main_queue, ^{
        /*
         *只能在主线程中执行的处理
         */
    });
});

```

---







