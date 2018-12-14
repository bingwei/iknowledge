# 任务与线程池

应将 **任务** （做什么）与 **执行策略** （怎么做）分离。

## 任务

### 定义任务

+ `Runnable` (since JDK 1.0)，无返回值，可传入 `execute()` 或 `submit()`
+ `Callable` (since JDK 1.5)，有返回值，可传入 `submit()`

### 提交任务

+ `execute` (来自 `Executor`)，用来执行 `Runnable`
+ `submit` (来自 `ExecutorService`)，用来执行 `Runnable` / `Callable`，返回 `Future`

### 任务执行结果

`Future`

+ Blocking operation

## 线程池

`ExecutorService` 的子类。

线程池使用 blocking queue 来管理进来的任务。

线程池中线程的合理数量取决于实际的任务类型。对于 CPU-intensive 的任务，一般设置和 CPU 核心数量一样的线程数；对于 IO-intensive 的任务，线程的数量就可以比较多。

标准库中内置的几种线程池（调用 `Executors` 中的静态工厂生成）：

+ Fixed thread pool
  + 固定数量的线程
+ Cached thread pool
  + 线程数量不限制
  + 会回收 idle 线程
+ Single thread executor
  + 容量为 1 的 fixed thread pool
  + 能保证任务顺序执行
+ Scheduled thread pool
  + 以一定的策略调度任务
  + 使用三种 `scheduleXxx` 方法来定义调度方案
+ Single thread scheduled executor

## 线程池的执行策略

线程池的执行策略包括：

+ 任务运行在哪个线程中
+ 任务运行的顺序（FIFO，LIFO，或是基于优先级）
+ 同时运行的任务数量
+ 排队等待的任务数量
+ 系统过载时选择放弃哪个任务，如何通知这件事
+ 任务执行前后需要做哪些处理

`Executors.newXxxThreadPool` 都是调用 `new ThreadPoolExecutor(...)`，传递不同的参数来构建几个默认情况下的线程池。`ThreadPoolExecutor` 接收的参数为：

+ `corePoolSize: int` 线程数量下限/基准数量
+ `maxPoolSize: int` 线程数量上限
+ `keepAliveTime: long` + `unit: TimeUnit` 等待 idle 线程多久时间
+ `workQueue: BlockingQueue<Runnable>` 任务等待队列
+ `threadFactory: ThreadFactory` thread factory，默认为 `DefaultThreadFactory`
+ `handler: RejectedExecutionHandler` 任务无法提交至队列时的 callback，默认为 abort policy (抛出异常)

| 线程池 | `corePoolSize` | `maxPoolSize` | `keepAliveTime` | `workQueue` |
| :-: | :-: | :-: | :-: | :-: |
| Fixed thread pool | `n` | `n` | N/A | `LinkedBlockingQueue` |
| Cached thread pool | 0 | `Integer.MAX_VALUE` | 60 secs | `SynchronousQueue` |
| Single thread executor | 1 | 1 | N/A | `LinkedBlockingQueue` |
| Scheduled thread pool | `n` | `Integer.MAX_VALUE` | 60 secs | `DelayedWorkQueue` |

Fixed thread pool 和 single thread executor 的线程数量是固定的，因此使用可以无限增长的 `LinkedBlockingQueue` 作为任务队列；cached thread pool 的线程数量没有上限，因此不需要在队列中存储任务，使用无容量的 `SynchronousQueue`。

`RejectedExecutionHandler` 说明了当任务队列已满时，使用何种策略进行处理。标准库中提供了四个默认的策略（源码在 `ThreadPoolExecutor` 中，都非常简单）。当新任务来临时：

+ `AbortPolicy` 抛出异常 `RejectedExecutionException`
+ `DiscardPolicy` 新任务会被静默地丢掉
+ `DiscardOldestPolicy` 新任务挤掉一个最老的任务
+ `CallerRunsPolicy` 在 caller 的线程中执行任务

## 带调度的线程池

`ScheduledExecutorService` 是 `ExecutorService` 的子类。

## 生命周期

+ Shundown
  + `shutdown()`
  + `awaitTermination()`
  + `shutdownNow()`
+ Test
  + `isShutdown()`
  + `isTerminated()`

## 取消任务

+ 不要使用 `Thread::stop`
+ [处理 `InterruptedException`](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html)