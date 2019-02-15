# Java Concurrency

## Roadmap

+ _Java Concurrency in Practice_ （[豆瓣](https://book.douban.com/subject/1888733/)）建议看英文版
+ [《深入浅出 Java Concurrency》系列文章](http://www.blogjava.net/xylz/archive/2010/07/08/325587.html)
+ [Java Tutorials: Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)

## Contents

+ [原子性/线程安全](concurrency-threadsafety.md)
+ [任务与线程池](concurrency-threadpool.md)
+ [锁](concurrency-lock.md)
+ [同步机制](concurrency-sync.md)
+ [并发集合](concurrency-collection.md)

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
+ [JavaDoop 博客](https://javadoop.com/)