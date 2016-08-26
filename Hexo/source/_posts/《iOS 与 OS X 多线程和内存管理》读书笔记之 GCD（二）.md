---
title: 《iOS 与 OS X 多线程和内存管理》读书笔记之 GCD（二）
date: 2016-08-16 11:44:41
tags: [iOS, GCD]
categories: "iOS"
description: "第二篇 GCD 笔记 : )"
---

> [第一篇基础篇 👈](/2016/08/16/《iOS 与 OS X 多线程和内存管理》读书笔记之 GCD（一）/)

---

## dispatch_set_target_queue

> 作用：变更生成的 Dispatch Queue 的执行优先级。


```objc
dispatch_queue_t targetQueue = dispatch_queue_create("com.iBinaryOrg.targetQueue", DISPATCH_QUEUE_SERIAL);

dispatch_queue_t queue1 = dispatch_queue_create("queue1", DISPATCH_QUEUE_SERIAL);
//设置优先级
dispatch_set_target_queue(queue1, targetQueue);
```

dispatch_set_target_queue 中第一个参数为指定要变更执行优先级的 Dispatch Queue，第二个参数为指定的与要使用的执行优先级相同优先级的 Dispatch Queue。

如上面代码中所示，将 Dispatch Queue 指定为 dispatch_set_target_queue 函数的参数，不仅可以变更 Dispatch Queue 的执行优先级，还可以作为 Dispatch Queue 的执行阶层。如果在多个 Serial Dispatch Queue 中用 dispatch_set_target_queue 函数指定目标为某一个 Serial Dispatch Queue，那么原先本应该并行执行的多个 Serial Dispatch Queue，在目标 Serial Dispatch Queue 上只能同时执行一个处理。

> 在必须将不可并行执行的处理追加到多个 Serial Dispatch Queue 中时，如果使用 dispatch_set_target_queue 函数将目标指定为某一个 Serial Dispatch Queue，即可防止处理并行执行。

比如：

```objc
- (void)testTargetQueue {
    dispatch_queue_t targetQueue = dispatch_queue_create("com.iBinaryOrg.targetQueue", DISPATCH_QUEUE_SERIAL);
    
    //创建 3 个同步队列
    dispatch_queue_t queue1 = dispatch_queue_create("queue1", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t queue2 = dispatch_queue_create("queue2", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t queue3 = dispatch_queue_create("queue3", DISPATCH_QUEUE_SERIAL);

    //设置优先级(标记 1)
    dispatch_set_target_queue(queue1, targetQueue);
    dispatch_set_target_queue(queue2, targetQueue);
    dispatch_set_target_queue(queue3, targetQueue);
	
    dispatch_async(targetQueue, ^{
        NSLog(@"target queue in");
        [NSThread sleepForTimeInterval:4];
        NSLog(@"target queue out");
    });

    dispatch_async(queue1, ^{
        NSLog(@"1 in");
        [NSThread sleepForTimeInterval:3];
        NSLog(@"1 out");
    });

    dispatch_async(queue2, ^{
        NSLog(@"2 in");
        [NSThread sleepForTimeInterval:2];
        NSLog(@"2 out");
    });

    dispatch_async(queue3, ^{
        NSLog(@"3 in");
        [NSThread sleepForTimeInterval:1];
        NSLog(@"3 out");
    });
}
```

输出：

```
2016-05-27 15:31:24.630 GCD_Demo[2540:25b] target queue in
2016-05-27 15:31:28.634 GCD_Demo[2540:25b] target queue out
2016-05-27 15:31:28.636 GCD_Demo[2540:25b] 1 in
2016-05-27 15:31:31.640 GCD_Demo[2540:25b] 1 out
2016-05-27 15:31:31.642 GCD_Demo[2540:25b] 2 in
2016-05-27 15:31:33.645 GCD_Demo[2540:25b] 2 out
2016-05-27 15:31:33.648 GCD_Demo[2540:25b] 3 in
2016-05-27 15:31:34.651 GCD_Demo[2540:25b] 3 out
```

以上的代码中，如果将（标记 1）处的 `dispatch_set_target_queue` 三句代码注释掉，则会出现以下结果：

```
2016-05-27 14:34:40.245 GCD_Demo[1796:1403] target queue in
2016-05-27 14:34:40.246 GCD_Demo[1796:21b] 1 in
2016-05-27 14:34:40.247 GCD_Demo[1796:4203] 2 in
2016-05-27 14:34:40.247 GCD_Demo[1796:4303] 3 in
2016-05-27 14:34:41.253 GCD_Demo[1796:4303] 3 out
2016-05-27 14:34:42.253 GCD_Demo[1796:4203] 2 out
2016-05-27 14:34:43.249 GCD_Demo[1796:21b] 1 out
2016-05-27 14:34:44.249 GCD_Demo[1796:1403] target queue out
```

> 正如在 [dispatch_queue_create](/2016/08/16/《iOS 与 OS X 多线程和内存管理》读书笔记之 GCD（一）/#dispatch-queue-create) 小节中所说的，虽然 queue1、queue2 和 queue3 是串行队列，但由于将不同的 3 个 block 分别追加到了这 3 个串行队列中，所以这 3 个串行队列是`同时处理`的。

---

## dispatch_after

```objc
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC));
dispatch_after(time, dispatch_get_main_queue(), ^{
    NSLog(@"waited at least three seconds.");
});

```

> 注意：dispatch_after 函数并不是在指定时间后执行处理，而只是在指定时间追加处理到 Dispatch Queue。此源码与在 3 秒后用 dispatch_async 函数追加 Block 到 Main Dispatch Queue 的相同。

第二个参数指定要追加处理的 Dispatch Queue，第三个参数指定记述要执行处理的 Block。

---

## Dispatch Group

```objc
- (void)testDispatchGroup {
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();
    
    //将队列添加进 group 中
    dispatch_group_async(group, queue, ^{NSLog(@"blk0");});
    dispatch_group_async(group, queue, ^{NSLog(@"blk1");});
    dispatch_group_async(group, queue, ^{NSLog(@"blk2");});
    
    dispatch_group_notify(group, queue, ^{NSLog(@"done");});
}

```
执行结果如下：

```
blk0
blk2
blk1
done
```

`dispatch_group_notify` ：用来监听队列中的任务已全部执行完毕，第三个参数 `block`用来执行 group 中任务完毕所响应的操作。

另外，在 Dispatch Group 中也可以使用 `dispatch_group_wait` 函数仅等待全部处理执行结束。

```objc
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);//第二个参数指定为等待的时间（超时），此处意味着永久等待。**只要属于 Dispatch Group 的处理尚未执行结束，就会一直等待，中途不能取消**
```

如果想指定等待时间间隔为 1 秒时应做如下处理：

```objc
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC));
long result = dispatch_group_wait(group, time);

if (0 == result) {
    /**
     *  属于 Dispatch Group 的全部处理执行结束
     */
} else {
    /**
     *  属于 Dispatch Group de 某一个处理还在执行中
     */
}

```

> 当等待时间为 DISPATCH_TIME_FOREVER、由 dispatch_group_wait 函数返回时，由于属于 Dispatch Group 的处理必定全部执行结束，所以返回值恒定为 0.
> 这里的“等待”是什么意思呢？这意味着一旦调用 dispatch_group_wait 函数，该函数就处于调用的状态而不返回。即执行 dispatch_group_wait 函数的现在的线程（当前线程）停止。在经过 dispatch_group_wait 函数中指定的时间或属于指定 Dispatch Queue 的处理全部执行结束前，执行该函数的线程停止。

指定 DISPATCH_TIME_NOW，则不用任何等待即可判定属于 Dispatch Group 的处理是否执行结束。

```objc
long result = dispatch_group_wait(group, DISPATCH_TIME_NOW);
```

在主线程的 RunLoop 的每次循环中，可检查执行是否结束，从而不耗费多余的等待时间，虽然这样也可以，但一般在这种情况下，还是推荐用 `dispatch_group_notify` 函数追加结束处理到 Main Dispatch Queue 中。这是因为 `dispatch_group_notify` 函数可以简化源代码。


---

## dispatch_barrier_async

举例：

```objc
- (void)testDispatchBarrierAsync {
	// 创建并行队列
    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.gcd.ForBarrier", DISPATCH_QUEUE_CONCURRENT);
    
    // 异步执行并行队列
    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk0_for_reading");
    });
    
    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk1_for_reading");
    });
    
    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk2_for_reading");
    });
    
    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk3_for_reading");
    });
	
    // 此处会等待已经追加到 concurrentQueue 中的 block 操作全部执行完成，
    // 再调用 dispatch_barrier_async 函数。
    dispatch_barrier_async(concurrentQueue, ^{
        NSLog(@"blk_for_writting");
    });

	// 继续异步执行并行队列
    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk4_for_reading");
    });

    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk5_for_reading");
    });

    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk6_for_reading");
    });
    
    dispatch_async(concurrentQueue, ^{
        NSLog(@"blk7_for_reading");
    });
}

```

> dispatch_barrier_async 函数会等待追加到 Concurrent Dispatch Queue 上的并行执行的处理全部结束之后，再将指定的处理追加到该 Concurrent Dispatch Queue 中。然后再由 dispatch_barrier_async 函数追加的处理执行完毕后，Concurrent Dispatch Queue 才恢复为一般的动作，追加到该 Concurrent Dispatch Queue 的处理又开始并行执行。

输出：

```
2016-05-27 16:12:41.103 GCD_Demo[2540:4507] blk1_for_reading
2016-05-27 16:12:41.102 GCD_Demo[2540:43d7] blk0_for_reading
2016-05-27 16:12:41.105 GCD_Demo[2540:43d7] blk3_for_reading
2016-05-27 16:12:41.106 GCD_Demo[2540:4507] blk2_for_reading
2016-05-27 16:12:41.108 GCD_Demo[2540:43d7] blk_for_writting //<--
2016-05-27 16:12:41.109 GCD_Demo[2540:43d7] blk4_for_reading
2016-05-27 16:12:41.112 GCD_Demo[2540:43d7] blk6_for_reading
2016-05-27 16:12:41.109 GCD_Demo[2540:4507] blk5_for_reading
2016-05-27 16:12:41.113 GCD_Demo[2540:43d7] blk7_for_reading
```

*使用 Concurrent Dispatch Queue 和 dispatch_barrier_async 函数可实现 高效率 的数据库访问和文件访问。*


---

## dispatch_sync & 死锁问题

> 既然有“async”，当然也有“sync”，即 `dispatch_sync` 函数。它意味着“同步”（synchonous），也就是将指定的 Block “同步”追加到制定的 Dispatch Queue 中。在追加 Block 执行结束之前，dispatch_sync 函数会一直等待。
> **"等待" 意味着 `当前线程停止`。**

一旦调用 dispatch_sync 函数，那么在制定的处理执行结束之前，该函数不会返回。

### 死锁问题

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    dispatch_queue_t queue = dispatch_get_main_queue();
    dispatch_sync(queue, ^{NSLog@(@"Hello?");});   
}
```

该源代码在 Main Dispatch Queue 即主线程中执行制定 Block，并等待其执行结束。而其实主线程正在执行 `这些源代码`，所以 `无法执行` 追加到 Main Dispatch Queue 中的 Block。下面的例子也一样：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    dispatch_queue_t queue = dispatch_get_main();
    dispatch_async(queue, ^{
        dispatch_sync(queue, ^{NSLog(@"Hello?");});
    });
}
```

首先异步地向串行队列（main queue）中追加一个 block1，在这个 block1 中又同步地向此串行队列（main queue）追加一个 block2。所以按照串行队列的工作原理，需要等 block1 执行完毕，才会执行 block2，而 block1 执行的内容，就是在同样的串行队列中同步执行 block2，block2 又需要等待 block1 执行完毕之后再执行。所以此处形成了一个互相等待的情况，造成`死锁`。
此处的 dispatch_async 函数会返回，而 dispatch_sync 永远不会返回。

Serial Dispatch Queue 也会引起相同的问题。

```objc
dispatch_queue_t queue = dispatch_queue_create("com.gcd.serialDispatchQueue", NULL);
dispatch_async(queue, ^{
    dispatch_sync(queue, ^{NSLog(@"Hello?");});// 无法输出 “Hello?”
});
```

---

## dispatch_apply

dispatch_apply 函数是 dispatch_sync 函数和 Dispatch Group 的关联 API。该函数按指定的次数将指定的 Block 追加到指定的 Dispatch Queue 中，并等待全部处理执行结束。

```objc
dispatch_queue_t queue= dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(3, queue, ^(size_t index) {
    NSLog(@"%zu", index);
});
NSLog(@"done");
```

输出：

```
2
0
1
done
```

> 由于 dispatch_apply 函数也与 dispatch_sync 函数相同，会等待处理执行结束，因此推荐在 `dispatch_async` 函数中 **非同步** 地执行 dispatch_apply 函数。


---


## dispatch_suspend / dispatch_resume

挂起指定的 queue：

```objc
dispatch_suspend(queue);
```

恢复指定的 queue：

```objc
dispatch_resume(queue);
```

> 这些函数对已经执行的处理没有影响。挂起后，追加到 Dispatch Queue 中但尚未执行的处理在此之后停止执行。而恢复则使得这些处理能够继续执行。

---


## Dispatch Semaphore

Dispatch Semaphore 是持有计数的信号，该计数是多线程编程中的计数类型信号。所谓信号，类似于过马路时常用的手旗。可以通过时举起手旗，不可通过时放下手旗。而在 Dispatch Semaphore 中，使用计数来实现该功能。计数为 0 时等待，计数为 1 或大于 1 时。减去 1 而不等待。

使用方法：

```objc
dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
```

> 参数表示计数的初始值。例子中将计数值初始化为 1。


```objc
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
```

dispatch_semaphore_wait 函数等待 Dispatch Semaphore 的计数值达到大于或等于 1.当计数值大于等于 1，或者在待机中计数值大于等于 1 时，对该计数进行减法并从 dispatch_semaphore_wait 函数返回。第二个函数指定等待时间。

```objc
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC));
long result = dispatch_semaphore_wait(semaphore, time);

if (0 == result) {

    /*
     * 由于 Dispatch Semaphore 的计数值达到大于等于 1
     * 或者在待机中的指定时间内
     * Dispatch Semaphore 的计数值达到大于等于 1
     * 所以 Dispatch Semaphore 的计数值减去 1
     *
     * 可执行需要进行排他控制的锤炼
     */
     
} else {

    /*
     * 由于 Dispatch Semaphore 的计数值为 0
     * 因此在达到指定时间为止待机
     */
     
}
```

dispatch_semaphore_wait 函数返回 0 时，可安全地执行需要进行排他控制的处理。
该处理结束时 **`通过 dispatch_semaphore_signal 函数将 Dispatch Semaphore 的计数值加 1`**.

Sample:

```objc
- (void)testDispatchSemaphore {
	//创建一个 semaphore
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    
    __block NSString *strTest = @"test";

    dispatch_queue_t concurrentQueue = dispatch_queue_create("com.gcd.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_async(concurrentQueue, ^{
    	
    	//每次执行之前，先判定信号量是否大于 0，是的话就执行，否则等待。
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        
        if ([strTest isEqualToString:@"test"]) {
        
            NSLog(@"--%@--1--", strTest);
            [NSThread sleepForTimeInterval:1];
            
            if ([strTest isEqualToString:@"test"]) {
                NSLog(@"--%@--2--", strTest);
                [NSThread sleepForTimeInterval:1];                
            } else  {
                NSLog(@"==========changed========");
            }
        }
        //执行完毕之后，信号量进行 +1 处理
        dispatch_semaphore_signal(semaphore);
    });

    dispatch_async(concurrentQueue, ^{
    
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

        NSLog(@"--%@--3--", strTest);
        
        [NSThread sleepForTimeInterval:1];
        dispatch_semaphore_signal(semaphore);
        
    });

    

    dispatch_async(concurrentQueue, ^{

        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

        strTest = @"modify";
        NSLog(@"--%@--4--", strTest);

        [NSThread sleepForTimeInterval:1];
        dispatch_semaphore_signal(semaphore);

    });

    

    dispatch_async(concurrentQueue, ^{

        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);

        NSLog(@"--%@--5--", strTest);
        [NSThread sleepForTimeInterval:1];

        dispatch_semaphore_signal(semaphore);

    });
}
```

输出：

```
2016-05-27 17:15:39.490 GCD_Demo[2540:4717] --test--1--
2016-05-27 17:15:40.494 GCD_Demo[2540:4717] --test--2--
2016-05-27 17:15:41.498 GCD_Demo[2540:4517] --test--3--
2016-05-27 17:15:42.502 GCD_Demo[2540:442f] --modify--4--
2016-05-27 17:15:43.505 GCD_Demo[2540:4283] --modify--5--
```

> 解析：以上虽然 Block 都是在异步队列中执行，但是由于初始化 Semaphore 时是以 1 的信号量创建的，所以每一个 Block 在追加到异步队列中之后，由于执行之前都通过了信号量的判断，所以同时只有一个 Block 能够执行。直到一个 Block 执行完毕，恢复了信号量之后，下一个 Block 才能得以执行。


---


## dispatch_once

dispatch_once 函数是保证在应用程序执行中只执行一次指定处理的API。

```objc
static dispatch_once_t pred;
dispatch_once(&pred, ^{
    //初始化
});
```

> **单例模式**，此源代码能够保证即使在多线程环境下执行，也是百分之百安全。

---


## Dispatch I/O

（待完善）

---

