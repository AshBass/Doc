# iOS多线程 —— GCD
    官方[开源代码](http://libdispatch.macosforge.org)

***

## 前言

### 线程 、任务 、队列
>   1. 线程 程序执行任务的最小调度单位
>   2. 任务 你所要做的事情 ，说白了就是你的代码
>   3. 队列 用来存放任务的数组

### 串行 、并行 、并发
>   1. 串行 串行程序中，程序会按照顺序执行每一条指令，在整个程序运行过程中，仅存在一个运行上下文。（即一个调用栈，一个堆）
>   2. 并发 并发指的是，程序在运行中存在多个运行上下文，对应不同的调用栈。
>   3. 并行 在只有一个处理器的机器上，并发程序虽然有多个运行上下文，但某一个时刻只有一个任务在运行。在多处理器上，则可以有多个。 
>   同一时刻有多个任务同时运行的方式就是并发。
>   4. 关于 并行 与 并发 :
>   你吃饭吃到一半，电话来了，你一直到吃完了以后才去接，这就说明你不支持并发也不支持并行。
>   你吃饭吃到一半，电话来了，你停了下来接了电话，接完后继续吃饭，这说明你支持并发。
>   你吃饭吃到一半，电话来了，你一边打电话一边吃饭，这说明你支持并行。
>   并发的关键是你有处理多个任务的能力，不一定要同时。并行的关键是你有同时处理多个任务的能力。最关键的点就是：是否是『同时』。

### 同步 、异步
>   1. 同步 不具备创建新线程的能力，所有的任务都要等上一个任务完成才能执行
>   2. 异步 具备创建新线程的能力，任务创建后可以先绕过，回头再执行

### 串行队列 、 并行队列
>   1. 串行队列 队列中的任务要按顺序执行
>   2. 并行队列 队列中的任务可同时执行

## 组合
|   类型  | 串行队列    |   并行队列    |
| :-----: | :-----: | :-----: |
| 同步执行  | 不开启新的线程，任务按顺序进行    |   不开启新的线程，任务按顺序进行  |
| 异步执行  | 开启一个新线程，任务按顺序进行    |   开启多个新线程，多个任务同时进行    |

***

## 正文

### GCD概要
>   1. GCD和operation queue一样都是基于队列的并发编程API，他们通过集中管理大家协同使用的线程池。
>   2. 公开的5个不同队列：运行在主线程中的main queue，3个不同优先级的后台队列（High Priority Queue，Default Priority Queue，Low Priority Queue），以及一个优先级更低的后台队列Background Priority Queue（用于I/O）
>   3. 可创建自定义队列：串行或并列队列。自定义一般放在Default Priority Queue和Main Queue里。
>   4. 操作是在多线程上还是单线程主要是看队列的类型和执行方法，并行队列异步执行才能在多线程，并行队列同步执行就只会在主线程执行了

### 系统队列
>   1. 主队列，主线程唯一队列，一个串行队列
    通过 dispatch_get_main_queue() 可获得主队列
>   2. 全局队列，一个并行的队列

```
参数1   identifier : 优先级
    - DISPATCH_QUEUE_PRIORITY_HIGH          : 用户发起需要马上得到结果进行后续任务
    - DISPATCH_QUEUE_PRIORITY_DEFAULT       : 默认的不应该使用这个设置任务
    - DISPATCH_QUEUE_PRIORITY_LOW           : 花费时间稍多比如下载，需要几秒或几分钟的
    - DISPATCH_QUEUE_PRIORITY_BACKGROUND    : 不可见在后台的操作可能需要好几分钟甚至几小时的
参数2   flags : 保留以备将来使用。传递除零值以外的任何值可能导致空返回值。一般传0就可以

dipatch_queue_t queue = dispatch_get_global_queue(long identifier, unsigned long flags)

```

### 自定义队列 dispatch queue
```
参数1   label : 要附加到队列的字符串标签。此参数是可选的，可能为null。
参数2   attr : 队列属性，填入NULL默认为串行队列（DISPATCH_QUEUE_SERIAL）
    - DISPATCH_QUEUE_SERIAL     :   串行队列
    - DISPATCH_QUEUE_CONCURRENT :   并行队列

dispatch_queue_t queue = dispatch_queue_create(const char *_Nullable label, dispatch_queue_attr_t _Nullable attr)

```

### 自定义队列优先级 dispatch_queue_attr_t
```
参数1   attr : 队列属性，填入NULL默认为串行队列（DISPATCH_QUEUE_SERIAL）
    - DISPATCH_QUEUE_SERIAL     :   串行队列
    - DISPATCH_QUEUE_CONCURRENT :   并行队列
参数2   qos_class : 队列优先级，按下面表格填入对应的优先级
参数3   relative_priority : QOS类中填入相对优先级，取值为 -15 ～ 0 之间，否则会返回空值NULL

dispatch_queue_attr_t attr = dispatch_queue_attr_make_with_qos_class(dispatch_queue_attr_t _Nullable attr, dispatch_qos_class_t qos_class, int relative_priority);
dispatch_queue_t queue = dispatch_queue_create("com.xunxintech.queue", attr);
```

#### qos_class 对应关系如下
|   DISPATCH_QUEUE_PRIORITY  |  QOS_CLASS   |
|  :-----: | :-----: |
|   DISPATCH_QUEUE_PRIORITY_HIGH  |   QOS_CLASS_USER_INITIATED  |
|   DISPATCH_QUEUE_PRIORITY_DEFAULT  |   QOS_CLASS_DEFAULT  |
|   DISPATCH_QUEUE_PRIORITY_LOW  |   QOS_CLASS_UTILITY  |
|   DISPATCH_QUEUE_PRIORITY_BACKGROUND  |   QOS_CLASS_BACKGROUND   |


### 关于 QOS_Class 、队列类型

| queue | QOS_CLASS | 应用场景 |
|  :-----: | :-----: | :-----: |
|   Main Thread  |   QOS_CLASS_USER_INTERACTIVE  | 表示任务需要被立即执行提供好的体验，用来更新UI，响应事件等。这个等级最好保持小规模。 |
|   DISPATCH_QUEUE_PRIORITY_HIGH  |   QOS_CLASS_USER_INITIATED  | 表示任务由UI发起异步执行。适用场景是需要及时结果同时又可以继续交互的时候 。|
|   DISPATCH_QUEUE_PRIORITY_DEFAULT  |   QOS_CLASS_DEFAULT  | 默认的不应该使用这个设置任务 |
|   DISPATCH_QUEUE_PRIORITY_LOW  |   QOS_CLASS_UTILITY  | 表示需要长时间运行的任务，伴有用户可见进度指示器。经常会用来做计算，I/O，网络，持续的数据填充等任务。这个任务节能。 |
|   DISPATCH_QUEUE_PRIORITY_BACKGROUND  |   QOS_CLASS_BACKGROUND   | 表示用户不会察觉的任务，使用它来处理预加载，或者不需要用户交互和对时间不敏感的任务。 |

### dispatch_once
```
static dispatch_once_t onceToken;
     dispatch_once(&onceToken, ^{
          //这里面的代码只会执行一次，直到程序结束
     });
```

### dispatch_async
```
开启一个异步线程，把任务传入队列中执行
dispatch_async(xunxintechQueue, ^(void){
    //要执行的任务
});
```

### dispatch_after
```
延时执行一个任务

//参数1 ： 填入 DISPATCH_TIME_NOW 表示从当前时间开始延后
//参数2 ： 填入纳秒数值。NSEC_PER_SEC 表示每秒有多少纳秒。 2 * NSEC_PER_SEC 表示2秒
dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, (int64_t) (2 * NSEC_PER_SEC));

dispatch_after(time, dispatch_get_main_queue(), ^(void){
    //要执行的任务
});
```

### dispatch_apply
系统创建异步线程是有数量限制的，所以，如果在for循环中创建过多的异步线程，可能会导致程序死锁，造成卡顿。
我们可以用 dispatch_apply 来避免这个问题。
```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.xunxintech.queue",DISPATCH_QUEUE_CONCURRENT);

//有问题的情况，可能会死锁
for (int i = 0; i < 999 ; i++) {
    dispatch_async(concurrentQueue, ^{
        //任务
    });
}

//会优化很多，能够利用GCD管理
dispatch_apply(999, concurrentQueue, ^(size_t i){
    //任务
});
```

### dispatch_group
dispatch groups是专门用来监视多个异步任务。dispatch_group_t实例用来追踪不同队列中的不同任务。

1.dispatch_group_wait，会阻塞当前进程，等所有任务都完成或等待超时
```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.xunxintech.queue",DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t group = dispatch_group_create();
//在group中添加队列的block
dispatch_group_async(group, concurrentQueue, ^{
    [NSThread sleepForTimeInterval:2.f];
    NSLog(@"第一个任务完成");
});
dispatch_group_async(group, concurrentQueue, ^{
    NSLog(@"第二个任务完成");
});
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
NSLog(@"所有任务已经完成");
```

2.使用dispatch_group_notify，异步执行闭包，不会阻塞
```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.xunxintech.queue",DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, concurrentQueue, ^{
    NSLog(@"第一个任务完成");
});
dispatch_group_async(group, concurrentQueue, ^{
    NSLog(@"第二个任务完成");
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"所有任务已经完成");
});
NSLog(@"这里不阻塞，继续执行");
```

3.使用 dispatch_group_enter 和 dispatch_group_leave
```
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
NSLog(@"group one start");
dispatch_group_enter(group);
dispatch_async(queue, ^{
    sleep(1); //模拟异步请求
    NSLog(@"任务完成");
    dispatch_group_leave(group);
});

dispatch_group_notify(group, queue, ^{
    NSLog(@"所有任务完成");
});
```

### dispatch_barrier_sync 和 dispatch_barrier_async
#### 相同点
1、等待在它前面插入队列的任务先执行完
2、等待他们自己的任务执行完再执行后面的任务
#### 不同点
dispatch_barrier_sync 会阻塞当前线程，dispatch_barrier_async 则不会

```
dispatch_queue_t concurrentQueue = dispatch_queue_create("com.xunxintech.queue",DISPATCH_QUEUE_CONCURRENT);

dispatch_async(concurrentQueue, ^{
    NSLog(@"任务1);
});
dispatch_barrier_async(concurrentQueue, ^{
    [NSThread sleepForTimeInterval:1.0];
    NSLog(@"我是barrier");
});
NSLog(@"执行了 barrier);
dispatch_async(concurrentQueue, ^{
    NSLog(@"任务3);
});
```

以上代码的输出为：
```
任务1
执行了 barrier
我是barrier
任务3
```

### dispatch_semaphore
使用 dispatch_semaphore 可以保持线程的同步，有效解决资源抢占的问题
```
//创建 semaphore 设置为0
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    [NSThread sleepForTimeInterval:1.f];// 模拟耗时操作
    NSLog(@"semaphore + 1");
    dispatch_semaphore_signal(semaphore); //使 semaphore + 1
});
NSLog(@"semaphore - 1");
//执行 wait 会使 semaphore - 1 ，之后若 semaphore < 0 会阻塞当前线程
dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
NSLog(@"继续");
```

以上代码的输出为：
```
semaphore - 1
semaphore + 1
继续
```

### dispatch_source
用于监听系统的底层对象，主要处理的事件如下表：
|   方法  |  说明   |
|  :-----: | :-----: |
|   DISPATCH_SOURCE_TYPE_DATA_ADD  |   数据增加  |
|   DISPATCH_SOURCE_TYPE_DATA_OR  |   	数据OR  |
|   DISPATCH_SOURCE_TYPE_MACH_SEND  |   Mach端口发送  |
|   DISPATCH_SOURCE_TYPE_MACH_RECV  |   Mach端口接收   |
|   DISPATCH_SOURCE_TYPE_MEMORYPRESSURE  |   内存情况   |
|   DISPATCH_SOURCE_TYPE_PROC  |   进程事件   |
|   DISPATCH_SOURCE_TYPE_READ  |   读数据   |
|   DISPATCH_SOURCE_TYPE_SIGNAL  |   信号   |
|   DISPATCH_SOURCE_TYPE_TIMER  |   定时器   |
|   DISPATCH_SOURCE_TYPE_VNODE  |   文件系统变化   |
|   DISPATCH_SOURCE_TYPE_WRITE  |   文件写入   |

方法:
>> 1.dispatch_source_create：创建dispatch source，创建后会处于挂起状态进行事件接收，需要设置事件处理handler进行事件处理。
>> 2.dispatch_source_set_event_handler：设置事件处理handler
>> 3.dispatch_source_set_cancel_handler：事件取消handler，就是在dispatch source释放前做些清理的事。
>> 4.dispatch_source_cancel：关闭dispatch source，设置的事件处理handler不会被执行，已经执行的事件handler不会取消。

