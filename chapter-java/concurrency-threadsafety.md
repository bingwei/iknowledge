# Java Concurrency - 原子性与线程安全

## 线程安全 Thread safety

线程安全需要结合一个类的规约 (specification)。线程安全的类在任何的并发访问情况下都能满足其规约。

> A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of the execution of those threads by the runtime environment, and with no additional synchronization or other coordination on the part of the calling one. (Java Concurrency in Practice, Section 2.1)

几种线程安全的层次：

+ 不可变对象——绝对的线程安全
+ 完全线程安全
+ 不完全线程安全
  + e.g. `Vector`
+ 线程不安全

类中含有 _状态变量 (state variable)_，状态变量应当始终保持一致状态。如果有相互关联的状态变量，它们应当进行原子性的更新。否则就会出现 race condition。

编写正确的并发程序的重点就是管理**共享的**、**可变的**变量。

### Race condition 竞争条件

Race condition 的定义：多个线程并发执行，都访问并修改同一共享变量时（特别是复合操作 _read-modify-write_ 或 _check-then-act_），线程的调度顺序可能会导致程序输出错误的结果。那么我们说线程的调度顺序导致了 race condition，也就是多个线程相互“竞争”共享变量。

Data race 的定义：当两个线程中的指令访问同一个内存位置，且其中至少一个是写操作，并且没有同步操作控制先后顺序，则此处是一个 data race。

**Data race 和 race condition 是两个概念**。Data race 描述明确，可以自动化判断；而 race condition 是一个语义上的概念。很多 race condition 是由于 data race 导致的，但也不是所有的。Data race 也不一定会导致 race condition。

在编程中，我们应该关注的是语义层面上的 race condition。

参考：

+ [StackOverflow - Are “data races” and “race condition” actually the same thing in context of concurrent programming?](https://stackoverflow.com/questions/11276259/are-data-races-and-race-condition-actually-the-same-thing-in-context-of-conc)
+ [Race Condition vs. Data Race](https://blog.regehr.org/archives/490)

### Guard

为了避免 race condition，需要对共享变量的操作加锁。

正确的加锁方式是：(1) 每个访问该共享变量的地方都加锁；(2) 使用同一把锁。保证了这两点的情况下，则称这个锁 guard 了这个变量。这样可以保证在同一时刻只有一个线程访问(读/写)这个变量。

> For each mutable state variable that may be accessed by more than one thread, all accesses to that variable must be performed with the same lock held. In this case, we say that the variable is guarded by that lock.
> 对于一个可变的状态变量，如果每次对该变量的读/写都需要在拥有同一个特定的锁时进行，则称这个变量是 _guarded by_ 这个锁。

为了线程安全：

+ 任何一个共享的、可变的变量，都需要被一个锁所 guard
+ 任何一个涉及多个变量的 _不变式 (invariant)_，其中的所有变量都需要被一个锁所 guard

而且上述 guard 关系需要在文档中清晰地标注出来。

### Thread Confinement

Thread confinement 是一种保证线程安全的方式。如果将数据限制为只由一个线程进行访问，就可以天然避免 race condition，也不需要加锁。

实现方式：

+ 限制只有局部变量访问对象
  + 局部变量在栈上，天生就是限制在单个线程内的
+ 使用 `ThreadLocal`
  + 每个线程获得一个对象的拷贝，不共享对象
+ 其他方式
  + 很多 GUI 框架，例如 Swing 的 `invokeLater()`
  + JDBC 中，将 `Connection` 对象始终放在一个线程中执行

`ThreadLocal` 能同时实现：(1) 线程间的对象隔离；(2) 同一线程内方法间的对象共享。

`ThreadLocal` 的实现原理：在 `Thread` 类内部有一个 ThreadLocal -> object 的 map。

`ThreadLocal` 的典型使用场景：

+ 数据库连接：每个线程一个 `Connection` 对象
+ Session 管理：每个线程一个 `Session` 对象

## 原子性、可见性、顺序性

| | 原子性 | 可见性 | 顺序性 |
| :-: | :-: | :-: | :-: |
| `volatile` | | Y | Y |
| 锁 | Y | Y | Y |

### 原子性 Atomicity

原子性就是要保证：一系列的操作要么全部执行要么全部不执行。例如 `i++` 在 JVM 中实际上是三个操作：(1) 获取 `i` 的值；(2) 自增；(3) 再赋值给 `i`。

实现原子性的方法：

+ 使用 `AtomicLong` 等原子类
  + 有 `incrementAndGet()` 等特殊设计的方法
+ 对一系列操作加锁

### 可见性 Visibility

一个线程修改数据后，会首先更新缓存，之后才会更新主存。如果线程 A 修改的数据未更新到主存时，线程 B 读取此数据，就会读到陈旧的值 (_stale data_)。这就叫做线程 A 对数据的更改对 线程 B **不可见**。

实现可见性的方法：

+ 使用 `volatile` 修饰变量
  + 每次修改变量的值时，会立即更新到主存，并在其他缓存中清除该变量
+ 使用一把锁 guard 该变量

### 顺序性

JVM 为了运行效率，可能会进行指令重排，导致执行执行的顺序和代码的顺序不一样。在并发环境中，指令重排可能会导致数据不一致的问题（即不正确的优化）。

实现顺序性的方法：

+ 使用 `volatile` 修饰变量
+ 对修改变量才操作加锁（读变量的操作是否需要加锁？）
+ 隐式的顺序性保证：happens-before 原则

### volatile 的两个典型应用

注意：不应过于依赖 volatile。相比于锁，volatile 虽然使用方便，效率较高，但更难理解和推导，也更容易出错。

#### 应用一（可见性）：控制线程执行的状态变量

```Java
public class ExampleTask implements Runnable {

    private volatile boolean cancelled;

    public void run() {
        while (!cancelled) {
            compute();
        }
    }

    public void cancel() { cancelled = true; }
}
```

`volatile` 关键字可以保证某个线程调用 `cancel()` 更新 `cancelled` 变量时，变量更新的值可以刷新到主存中，被其他线程看到。保证线程及时停止。

#### 应用二（顺序性）：单例模式——双重检查锁

```Java
public class Singleton {
    // volatile 关键字是必须的，也是这个关键字的典型使用场景
    private volatile static Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        // 第一次 check，避免不必要的同步
        if (instance == null) {
            synchronized (Singleton.class) {
                // 第二次 check，避免重复实例化
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

语句 `instance = new Singleton();` 实际上是分为三个指令执行：(1) 分配内存空间；(2) 调用 constructor 初始化；(3) 将 instance 指针指向堆中内存地址。`volatile` 关键字可以防止这三条指令进行重排。如果 (3) 在 (2) 之前执行，某个线程可能会拿到未初始化的对象。

### 参考资料

+ [Java 多线程三大核心 - 原子性、可见性、顺序性](https://crossoverjie.top/JCSprout/#/thread/Threadcore)

## Java 内存模型 Memory Model

JMM 是跨平台的，JVM 负责处理平台特定的问题。

多线程程序不仅面临线程调度的不确定性，还可能出现指令重排。而 _happens-before_ 关系可以避免指令重排。

参考：

+ _Java Concurrency in Action_, Chapter 16

## Liveness problems

程序陷入某个无法跳出的状态，类似于单线程程序中的无限循环。这种问题也一般是在特定的线程调度下会触发。

+ Safety: "nothing bad ever happens"
+ Liveness: "something good eventually happens"

注意：单线程程序中也可能出现 safety problems，但 liveness problems 只可能在多线程程序中出现。

注意：liveness problems 不等于 deadlock，而是从属关系

### Deadlock 死锁

多个线程**互相等待**其他线程释放锁（系统资源），导致线程**无法改变其状态**，从而**永远等待**。

[哲学家就餐问题](https://zh.wikipedia.org/wiki/%E5%93%B2%E5%AD%A6%E5%AE%B6%E5%B0%B1%E9%A4%90%E9%97%AE%E9%A2%98)是一个典型的 deadlock 例子。

数据库系统可以检测并从死锁中恢复，而 Java 程序不可以。

Lock-ordering deadlock：多个线程都需要相同的一组锁，但是它们以不同的顺序获取锁。解决方法：对锁排序，程序中的所有线程都以固定的顺序获取锁，任何锁的顺序都是可行的，但是需要所有线程统一。

但是实际场景中，lock ordering 不是那么明显，可能两个相互协作的类会出现 lock ordering 的错误——在自己的 synchronized 方法中调用对方的 synchronized 方法。

在持有锁的情况下，调用可能阻塞的陌生方法，有死锁的风险。因此，应当限制上锁代码块的大小，尽量在不持有锁的情况下调用其他 method （称为 _open call_）。

死锁的发生当且仅当以下四个条件同时满足（_Coffman conditions_）：

+ 互斥 (mutual exclusion)：至少一种资源是以互斥的方式共享的，即任意时刻只能有一个线程使用该资源
+ 持有并等待 (hold and wait)：一个线程正持有至少一种资源，并正请求另一种其他线程持有的资源
+ 无特权 (no preemption)：资源只能由持有的线程自愿释放
+ 循环等待 (circular wait)：有一组线程，呈有向环的结构互相等待

### Livelock 活锁

与死锁类似，但线程的状态可以改变，线程**不断重试永远失败的操作**，从而**永远等待**。

Livelock 可以看作是 starvation 的特殊情况。

### Starvation 饥饿

一个线程**永远无法获得 (perpetually denied)** 需要的资源（通常是 CPU 周期），因而无法继续执行。

Starvation 通常是由不正确的**调度算法**导致，某些线程无法获得 CPU 时间。Java 中，如果随意设置线程优先级，可能导致饥饿问题。

Starvation 的弱化版本是 poor responsiveness。

Starvation 不一定是由互斥导致，资源泄漏或 DoS 攻击也可能导致 starvation。

## Performance problems

如果锁住的代码块过大（例如整个方法块），会导致同一时刻只能有一个线程执行该方法，多线程情况下可能会执行缓慢。要缓解这个问题，需要将大的 synchronized block 拆分为小的。Performance 和 simplicity 之间需要权衡。

TODO JCIP Chapter 11
