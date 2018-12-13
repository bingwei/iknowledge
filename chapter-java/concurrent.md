# Java Concurrent

## Roadmap

+ _Java Concurrency in Practice_ （[豆瓣](https://book.douban.com/subject/1888733/)）建议看英文版
+ [《深入浅出 Java Concurrency》系列文章](http://www.blogjava.net/xylz/archive/2010/07/08/325587.html)

可以分为五个部分的内容：

+ 原子性/线程安全
+ 任务 (Runnable, Callable) 与线程池 (executor)
+ 并发集合
+ 同步模式
+ 锁

TODO list：

+ ...

## 线程

Java 中的每个 `Thread` 都对应与操作系统的一个线程。线程创建的数量有上限。

## 任务与线程池

应将 **任务** （做什么）与 **执行策略** （怎么做）分离。

### 任务

+ `Runnable` (since JDK 1.0)，无返回值，可传入 `execute()` 或 `submit()`
+ `Callable` (since JDK 1.5)，有返回值，可传入 `submit()`

提交任务：

+ `execute` (来自 `Executor`)，用来执行 `Runnable`
+ `submit` (来自 `ExecutorService`)，用来执行 `Runnable` / `Callable`，返回 `Future`

任务执行结果：`Future`

+ Blocking operation

### 线程池

`ExecutorService` 的子类。

线程池使用 blocking queue 来管理进来的任务。

线程池中线程的合理数量取决于实际的任务类型。对于 CPU-intensive 的任务，一般设置和 CPU 核心数量一样的线程数；对于 IO-intensive 的任务，线程的数量就可以比较多。

`Executors` 中内置的几种线程池：

+ Fixed thread pool
  + 固定数量的线程
+ Cached thread pool
  + 使用 synchronized queue (容量为 0)
  + 线程数量不限制
  + 会回收 idle 线程
+ Single thread executor
  + 容量为 1 的 fixed thread pool
  + 能保证任务顺序执行
+ Scheduled thread pool
  + 使用 delay queue
  + 以一定的策略调度任务
  + 使用三种 `schedule...` 方法来定义调度方案
+ Single thread scheduled executor

### 线程池的执行策略

`Executors.newXxxThreadPool` 都是调用 `new ThreadPoolExecutor(...)`，传递不同的参数来构建几个默认情况下的线程池。`ThreadPoolExecutor` 接收的参数为：

+ `corePoolSize: int` 线程数量下限/基准数量
+ `maxPoolSize: int` 线程数量上限
+ `keepAliveTime: long` + `unit: TimeUnit` 等待 idle 线程多久时间
+ `workQueue: BlockingQueue<Runnable>` 任务等待队列
+ `threadFactory: ThreadFactory` thread factory
+ `handler: RejectedExecutionHandler` 任务无法提交至队列时的 callback

| 线程池 | `corePoolSize` | `maxPoolSize` | `keepAliveTime` | `keepAliveTime` |
| :-: | :-: | :-: | :-: | :-: |
| Fixed thread pool | `n` | `n` | N/A | `LinkedBlockingQueue` |
| Cached thread pool | 0 | `Integer.MAX_VALUE` | 60 secs | `SynchronousQueue` |
| Single thread executor | 1 | 1 | N/A | `LinkedBlockingQueue` |
| Scheduled thread pool | `n` | `Integer.MAX_VALUE` | 60 secs | `DelayedWorkQueue` |

Fixed thread pool 和 single thread executor 的线程数量是固定的，因此使用可以无限增长的 `LinkedBlockingQueue` 作为任务队列；cached thread pool 的线程数量没有上限，因此不需要在队列中存储任务，使用无容量的 `SynchronousQueue`。

不同的 `RejectedExecutionHandler` 策略下，当新任务来临时：

+ `AbortPolicy` 抛出异常 `RejectedExecutionException`
+ `DiscardPolicy` 新任务会被静默地丢掉
+ `DiscardOldestPolicy` 新任务挤掉一个最老的任务
+ `CallerRunsPolicy` 在 caller 的线程中执行任务

_Java Concurrency in Practice_ 6.2.2 中提到，线程池规定了六种执行策略：

+ 任务运行在哪个线程中
+ 任务运行的顺序（FIFO，LIFO，或是基于优先级）
+ 同时运行的任务数量
+ 排队等待的任务数量
+ 系统过载时选择放弃哪个任务，如何通知这件事
+ 任务执行前后需要做哪些处理

### 生命周期

+ Shundown
  + `shutdown()`
  + `awaitTermination()`
  + `shutdownNow()`
+ Test
  + `isShutdown()`
  + `isTerminated()`

### 取消任务

+ 不要使用 `Thread::stop`
+ [处理 `InterruptedException`](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html)

## 原子性与线程安全

### volatile

`volatile` 关键字是关于 visibility 的。

+ 防止编译器进行不正确的优化（例如优化成常量）
+ 每一次变量值的修改都直接写到主存中（而不是寄存器或 cache）

## 并发集合

### Unmodifiable view

+ `Collection.unmodifiableList()`
+ `Collection.unmodifiableMap()`
+ `Collection.unmodifiableSet()`

返回的都是一个 _unmodifiable view_。也就是说返回的对象是不可以修改的，但是对原对象的修改，view 对象可以看得到修改结果。

### Synchronized collections

+ `Collection.synchronizedList()`
+ `Collection.synchronizedMap()`
+ `Collection.synchronizedSet()`

在进行相互关联的操作的时候仍然要加锁：

```Java
List list = Collections.synchronizedList(new ArrayList());
    ...
synchronized (list) {
    Iterator i = list.iterator(); // Must be in synchronized block
    while (i.hasNext())
        foo(i.next());
}
```

这叫做 _client-side locking_。因为这些 synchronized collections 的内部实现是在所有的 method 上加 `synchronized`（即将锁加在 `this` 上），所以 `synchronized(list)` 将锁加在同样的对象上。

### Concurrent collections 

为并发而设计的集合，比 synchronized collections 要强大许多。

+ `ConcurrentMap` (`ConcurrentHashMap`, `ConcurrentSkipListMap`)
+ `BlockingQueue`
  + `LinkedBlockingQueue`
  + `ArrayBlockingQueue`
  + `SynchronousQueue`
+ `ConcurrentLinkedQueue`
+ `CopyOnWriteArrayList`
+ `CopyOnWriteArraySet`

### Copy-on-write (COW) 容器

+ `CopyOnWriteArrayList`
+ `CopyOnWriteArraySet`

## 同步模式

+ Blocking queue
+ Semaphore
+ Barrier
+ Latch

## Java 不同版本的并发编程设施变化

参考：[Java 多线程发展简史](https://raychase.iteye.com/blog/1679131)

### JDK 1.2

+ 废弃 `Thread::stop`, `Thread::suspend` 等方法
+ 引入 `ThreadLocal`

### JDK 5

+ 明确了内存模型
+ 基于 CAS 的 `java.util.concurrent` 包

带来的变化：

+ 此版本引入 `ScheduledThreadPoolExecutor` 之后，基本可以取代 `Timer` / `TimerTask`
+ 有了 `java.util.concurrent` 中的设施，基本上 wait / notify 已经完全用不到了

### JDK 6

+ 对锁的优化
+ 添加 `CyclicBarrier`

### JDK 7

+ 添加 fork/join 框架
