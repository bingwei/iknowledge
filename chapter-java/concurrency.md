# Java Concurrency

## Roadmap

+ _Java Concurrency in Practice_ （[豆瓣](https://book.douban.com/subject/1888733/)）建议看英文版
+ [《深入浅出 Java Concurrency》系列文章](http://www.blogjava.net/xylz/archive/2010/07/08/325587.html)
+ [Java Tutorials: Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)

可以分为五个部分的内容：

+ 原子性/线程安全
+ [任务与线程池](concurrency-threadpool.md)
+ [并发集合](concurrency-collection.md)
+ [同步机制](concurrency-sync.md)
+ [锁](concurrency-lock.md)

TODO list：

+ `StringBuilder` 和 `StringBuffer` 什么区别？ `StringBuffer` 是否是类似 `Vector` 的 synchronized collection?

## 线程

Java 中的每个 `Thread` 都对应与操作系统的一个线程。线程创建的数量有上限。

## 原子性与线程安全

原子性、可见性、顺序性：https://crossoverjie.top/JCSprout/#/thread/Threadcore

### volatile

`volatile` 关键字是关于 visibility 的。

+ 防止编译器进行不正确的优化（指令重排等）
+ 每一次变量值的修改都直接写到主存中（而不是寄存器或 cache）

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

## Java 不同版本的并发编程设施变化

参考：[Java 多线程发展简史](https://raychase.iteye.com/blog/1679131)

### JDK 1.2

+ 废弃 `Thread::stop`, `Thread::suspend` 等方法
+ 引入 `ThreadLocal`
+ 添加 `Collections.synchronizedXxx()` 方法

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

## 参考资料

+ [The Java Tutorials - Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)
+ _Java Concurrency in Practice_