# Java Concurrency - 锁

## 概念

### 互斥锁 (Mutex)

Mutex 即 _mutural exclusion lock_。只有一个线程可以拥有这个锁。

### 可重入锁 (Reentrant lock)

当一个线程请求一个它已经拥有的锁时，请求会成功。这样的锁叫做可重入锁。可重入锁意味着锁是每个线程分配一个，而不是每次调用分配一个。每个锁会有一个 count，每次 lock / unlock 时 +1 / -1。当 count 为零时锁会被释放。

`synchronized` 关键字和 `ReentrantLock` 都是可重入锁。如果 `synchronized` 没有可重入性，当子类重写了父类的一个 synchronized method 话，会导致 deadlock。

参考：_Java Concurrency in Practice_ 2.3.2

### Guarded by

对于一个可变的状态变量，如果每次对该变量的读/写都需要在拥有同一个特定的锁时进行，则称这个变量是 _guarded by_ 这个锁。这样可以保证在同一时刻只有一个线程访问(读/写)这个变量。

### 可中断锁

+ `synchronized` -- 不可中断锁
+ `ReentrantLock` -- 可中断锁

### 公平锁

+ `synchronized` -- 非公平锁
+ `ReentrantLock`, `ReentrantReadWriteLock` -- 可以设置为公平锁或非公平锁

## 锁的两个关键作用

无论是 `synchronized` 关键字，还是 `Lock` 都提供了两点作用：

+ Mutual exclusion
+ Memory visibility guarantee （见 _Java Concurrency in Practice_ 3.1 节和 16 章）

## `synchronized` 关键字

`synchronized` 关键字是一种 locking 机制，互斥锁，可重入锁。

_Synchronized methods_:

_Synchronized statements (block)_:

## `java.util.concurrent.locks` 中的锁

+ `Lock`
  + `ReentrantLock`
  + `ReadLock`
  + `WriteLock`
+ `ReadWriteLock`
  + `ReentrantReadWriteLock`

### `ReentrantLock`

`ReentrantLock`，互斥锁，可重入锁。`lock` 对应于进入 `synchronized` 块； `unlock` 对应于退出 `synchronized` 块。功能与 `synchronized` 类似，但是使用更加灵活。`ReentrantLock` 相比 `synchronized` 的优点：

+ 获取锁的过程不一定是阻塞的 (见 JCIP 13.1.1)
  + 使用 `tryLock`
  + 叫做 _timed lock acquisition_
  + 方便在有时间限制的任务中使用锁
+ 获取锁的过程是可中断的 (见 JCIP 13.1.2)
  + 使用 `lockInterruptibly`
  + 叫做 _interruptible lock acquisition_
  + 方便在可取消的任务中使用锁
+ 不必基于块结构 (synchronized block)
+ 可以控制多个线程同时读
+ 可以知道线程是否成功获取了锁

通常的使用模式：

```Java
Lock lock = new ReentrantLock();
...
lock.lock();
try {
    // update object state
} catch (...) {
    // restore invariants, if necessary
} finally {
    lock.unlock(); // 注意：必须在 finally 中释放锁，否则一旦出现异常，则无法释放锁
}
```

### `ReadWriteLock`

允许多个线程同时读

## CAS 与无锁数据结构

### CAS (compare and swap) 原子操作

基于 CPU 提供的原子操作指令实现，如 x86 的 `CMPXCHG` 指令。
CAS 是一种无锁的原子操作，用来实现 lock-free 的数据结构。

参考：

+ [Compare-and-swap - Wikipedia](https://en.wikipedia.org/wiki/Compare-and-swap)
+ [无锁队列的实现 - 酷壳](https://coolshell.cn/articles/8239.html)
