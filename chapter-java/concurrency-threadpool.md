# Java Concurrency - 任务与线程池

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

## 线程

Java 中的每个 `Thread` 都对应于操作系统的一个线程。线程创建的数量有上限。

线程的五种状态：

+ *新建 (new)*：新创建了一个线程对象
+ *可运行 (runnable)*：线程对象创建后，其他线程调用了该对象的 `start()` 方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取 CPU 的使用权
+ *运行 (running)*：Runnable 状态的线程获得了 CPU *时间片 (timeslice)*，执行程序代码
+ *阻塞 (block)*：阻塞状态是指线程因为某种原因放弃了 CPU 使用权，也即让出了 CPU timeslice，暂时停止运行。直到线程进入可运行 (runnable) 状态，才有 机会再次获得 cpu timeslice 转到 running 状态。阻塞的情况分三种：
  + *等待阻塞*：Running 状态的线程执行 `wait()`，JVM 会把该线程放入*等待队列 (waiting queue)* 中
  + *同步阻塞*：运行 (running) 的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则 JVM 会把该线程放入*锁池 (lock pool)* 中
  + *其他阻塞*： 运行 (running) 的线程执行 `Thread.sleep()` 或 `Thread.join()` 方法，或者发出了 I/O 请求时，JVM 会把该线程置为阻塞状态。当 `sleep()` 状态超时、`join()` 等待线程终止或者超时、或者 I/O 处理完毕时，线程重新转入 runnable 状态
+ *死亡 (dead)*：线程 `run()`、`main()` 方法执行结束，或者因异常退出了 `run()` 方法，则该线程结束生命周期。死亡的线程不可再次复生

![States of threads](concurrency/thread-states.jpg)

## 线程池

线程池一般是指 `ExecutorService` 的子类。`Executor` 接口定义了 `execute(Runnable)` 方法；`ExecutorService` 接口继承了 `Executor` 并定义了 submit, shutdown 等方法。

注意：**在《阿里巴巴 Java 开发手册》“并发处理” 这一章节，明确指出线程资源必须通过线程池提供，不允许在应用中自行显示创建线程。**



### 创建线程池

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

线程池中线程的合理数量取决于实际的任务类型。对于 CPU-intensive 的任务，一般设置和 CPU 核心数量一样的线程数；对于 IO-intensive 的任务，线程的数量就可以比较多。

此外，还可以直接调用 `ThreadPoolExecutor` 的 constructor 来创建线程池。

### 线程池的生命周期

+ Shundown
  + `shutdown()`
  + `awaitTermination()`
  + `shutdownNow()`
+ Test
  + `isShutdown()`
  + `isTerminated()`

### CompletionService

`CompletionService` 包装了 `ExecutorService`，增强了其功能。它维护一个 `Future` 对象的 blocking queue，称为 completion queue。当一个 future 中的任务完成之后，会放入 completion queue。这样调用 `CompletionService.take()` 取出 future 的时候，就不用再等待 future 了。（相当于一个 future 的多路选择？）

## 线程池的执行策略

### 线程池的内部结构

主要就两个结构：

+ 任务队列 work queue
+ Pool：线程池

提交一个任务之后，这个任务要么直接交给一个线程来运行，要么进入任务队列中等待。如果任务队列已满，则拒绝 (reject) 该任务。

任务运行在线程中。新添加的任务会放在任务队列中，任务队列可以决定任务运行的顺序（FIFO，LIFO，或是基于优先级）。任务从任务队列中取出，放在一个线程中执行。

线程池的执行策略包括：

+ 任务运行在哪个线程中
+ 任务运行的顺序（FIFO，LIFO，或是基于优先级）
+ 同时运行的任务数量
+ 排队等待的任务数量
+ 系统过载时选择放弃哪个任务，如何通知这件事
+ 任务执行前后需要做哪些处理

### ThreadPoolExecutor 的创建参数

`Executors` 中创建线程池的静态工厂都是调用了 `ThreadPoolExecutor` 的 constructor，并传递不同的参数。`ThreadPoolExecutor` 接收的参数为：

+ `corePoolSize: int` 线程数量下限/基准数量
+ `maxPoolSize: int` 线程数量上限
+ `keepAliveTime: long` 和 `unit: TimeUnit` 等待 idle 线程多久时间
+ `workQueue: BlockingQueue<Runnable>` 任务等待队列
+ `threadFactory: ThreadFactory` 线程工厂类
  + 默认为 `DefaultThreadFactory`，为线程设置可读的名字
+ `handler: RejectedExecutionHandler` 任务队列满时，如何拒绝任务
  + 默认为 abort policy (抛出异常)

| 线程池 | `corePoolSize` | `maxPoolSize` | `keepAliveTime` | `workQueue` |
| :-: | :-: | :-: | :-: | :-: |
| Fixed thread pool | `n` | `n` | N/A | `LinkedBlockingQueue` |
| Cached thread pool | 0 | `Integer.MAX_VALUE` | 60 secs | `SynchronousQueue` |
| Single thread executor | 1 | 1 | N/A | `LinkedBlockingQueue` |
| Scheduled thread pool | `n` | `Integer.MAX_VALUE` | 60 secs | `DelayedWorkQueue` |

Fixed thread pool 和 single thread executor 的线程数量是固定的，因此使用可以无限增长的 `LinkedBlockingQueue` 作为任务队列；cached thread pool 的线程数量没有上限，因此不需要在队列中存储任务，使用无容量的 `SynchronousQueue`。

### 行为设置

`RejectedExecutionHandler` 说明了当任务队列已满时，使用何种策略进行处理。标准库中提供了四个默认的策略（源码在 `ThreadPoolExecutor` 中，都非常简单）。当新任务来临时：

+ `AbortPolicy` 抛出异常 `RejectedExecutionException`
+ `DiscardPolicy` 新任务会被静默地丢掉
+ `DiscardOldestPolicy` 新任务挤掉一个最老的任务
+ `CallerRunsPolicy` 在 caller 的线程中执行任务

### 回调方法

+ `beforeExecute()`
+ `afterExecute()`
+ `terminated()`

用户可以继承 `ThreadPoolExecutor`，重写这些回调方法，来定义线程池的行为。

## 线程池的实现原理

### 类结构

继承关系：

`Executor` <- `ExecutorService` <- `AbstractExecutorService` <- `ThreadPoolExecutor`

`AbstractExecutorService` 实现了 `ExecutorService` 接口，提供了一些基本的实现。`submit(Callable)`（以及其他 `submit` 方法）提交上来的任务，会包装成一个 `FutureTask` （本身是 `Runnable` 的子类），委托给 `execute` 运行。

`ThreadPoolExecutor` 继承了 `AbstractExecutorService`，实现了线程池的各个功能。

### 线程池状态

`ThreadPoolExecutor.ctl` 存储线程池的 control state。它是一个 32 位的整数：

+ 高 3 位存放线程池状态 (run state)
  + 五个状态：running, shutdown, stop, tidying, terminated
+ 低 29 位存放线程数 (worker count)
  + 线程数量上限为 2^29 - 1，大约 5 亿

线程池状态与状态转移（五个状态只能按顺序转移，但可以跳过 shutdown 状态）：

+ Running: 可以接收新的任务，处理队列中的任务
  + 调用 `shutdown()`: -> shutdown
  + 调用 `shutdownNow()`: -> stop
+ Shutdown: 不接收新的任务，但处理队列中的任务
  + 调用 `shutdownNow()`: -> stop
  + 当 queue 和 pool 都清空后： -> tidying
+ Stop: 不接收新的任务，不处理队列中的任务，中断进行中的任务
  + 当 pool 清空后： -> tidying
+ Tidying: 所有任务已经结束，worker count = 0
  + 当回调方法 `terminated()` 结束： -> terminated
+ Terminated: 线程池全部停止，`awaitTermination()` 不再等待

### Worker

在线程池中，每个线程包装在一个 `Worker` 对象中，通过 `Worker` 对象管理线程的执行。

### 线程创建时机

+ 如果当前线程数少于 corePoolSize，那么提交任务的时候创建一个新的线程，并由这个线程执行这个任务
+ 如果当前线程数已经达到 corePoolSize，那么将提交的任务添加到队列中，等待线程池中的线程去队列中取任务
+ 如果队列已满，那么创建新的线程来执行任务，需要保证池中的线程数不会超过 maximumPoolSize，如果此时线程数超过了 maximumPoolSize，那么执行拒绝策略
  + 注意：对于无界队列，永远不满，那么线程数最多为 corePoolSize

### 异常处理

如果某个任务执行出现异常，那么执行任务的线程会被关闭，而不是继续接收其他任务。然后会启动一个新的线程来代替它。

### 什么时候任务会被拒绝

+ workers 的数量达到了 corePoolSize（任务此时需要进入任务队列），任务入队成功，与此同时线程池被关闭了，而且关闭线程池并没有将这个任务出队，那么执行拒绝策略。这里说的是非常边界的问题，入队和关闭线程池并发执行，读者仔细看看 execute 方法是怎么进到第一个 reject(command) 里面的。
+ workers 的数量大于等于 corePoolSize，将任务加入到任务队列，可是队列满了，任务入队失败，那么准备开启新的线程，可是线程数已经达到 maximumPoolSize，那么执行拒绝策略。

## 有延迟/周期的线程池

`ScheduledExecutorService` 是 `ExecutorService` 的子类，除了有一般线程池的功能，还可以**周期性**执行任务。它可以取代 `Timer` / `TimerTask` 的功能。

+ `schedule(Runnable/Callable<V> command, long delay, TimeUnit unit)`
  + 在指定的 delay 之后，运行任务
+ `scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`
  + 在指定的 initial delay 之后，每隔固定的 period 运行一次任务
+ `scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit)`
  + 在指定的 initial delay 之后，每次运行任务后，等待一个 delay 之后运行下一次任务

## 取消任务

`Thread::stop` 方法早已废弃，因为非常不安全。

```Java
try {
    SECONDS.sleep(1);
} finally {
    generator.cancel();
}
```

线程的 interruption 是一个信号，一个线程发送给另外一个线程的通知。没有任何规范定义 interruption 对应于“停止工作” (cancel) 的语义，但这已经约定俗成了。中断通知也没法保证停止目标线程的工作，它只是一个请求。

标准库里的**阻塞操作**都会抛出 `InterruptedException`。这个异常表示因为中断，执行提前终止。

调用 `Thread::interrupt` 中断一个线程，调用 `Thread::isInterrupted` 检查线程是否被中断。中断一个线程时，将会设置线程的 _中断状态 (interrupted status)_；如果线程正在执行阻塞操作，线程会抛出`InterruptedException`。

处理 `InterruptedException` 通常有两种方式：

+ 传播 exception
  + 直接在 method signature 上添加 `throws InterruptedException`，或者 catch 并 rethrow 这个异常
+ 存储异常状态，让上层代码可以处理
  + `Thread.currentThread().interrupt()`

例如，当 `ThreadPoolExecutor` 里的一个 worker 线程检测到中断消息——

+ 若线程池正在 shutdown，则进行一些线程池清理工作
+ 若线程池未 shutdown，则创建一个新线程，保证线程池的线程数正常

一般情况下，swallow 这个异常（catch 并什么都不做）是不合适的。

参考：

+ [处理 InterruptedException - developerworks](https://www.ibm.com/developerworks/cn/java/j-jtp05236.html)
+ [如何处理 InterruptedException - StackOverflow](https://stackoverflow.com/questions/3976344/handling-interruptedexception-in-java)