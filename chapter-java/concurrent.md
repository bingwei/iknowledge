# Java Concurrent

## Roadmap

+ _Java Concurrency in Practice_ （[豆瓣](https://book.douban.com/subject/1888733/)）建议看英文版
+ [《深入浅出 Java Concurrency》系列文章](http://www.blogjava.net/xylz/archive/2010/07/08/325587.html)
+ [Java Tutorials: Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)


可以分为五个部分的内容：

+ 原子性/线程安全
+ [任务与线程池](concurrent-threadpool.md)
+ 并发集合
+ 同步模式
+ 锁

TODO list：

+ ...

## 线程

Java 中的每个 `Thread` 都对应与操作系统的一个线程。线程创建的数量有上限。

## 原子性与线程安全

### volatile

`volatile` 关键字是关于 visibility 的。

+ 防止编译器进行不正确的优化（例如优化成常量）
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

注意 `Vector` 和 `Stack` 类早已过时了，现在应该用 `ArrayList` 和 `Deque` 配合 `Collections.synchronizedXxx`。

## 同步模式

+ Blocking queue
+ Semaphore
+ Barrier
+ Latch

## Misc

### CAS (compare and swap) 原子操作

基于 CPU 提供的原子操作指令实现，如 x86 的 `CMPXCHG` 指令。
CAS 是一种无锁的原子操作，用来实现 lock-free 的数据结构。

参考：

+ [Compare-and-swap - Wikipedia](https://en.wikipedia.org/wiki/Compare-and-swap)
+ [无锁队列的实现 - 酷壳](https://coolshell.cn/articles/8239.html)

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
