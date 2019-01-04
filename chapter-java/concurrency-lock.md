# Java Concurrency - 锁 

## 概念

### 互斥锁 (Mutex)

Mutex 即 _mutural exclusion lock_。只有一个线程可以拥有这个锁。

### 可重入锁 (Reentrant lock)

当一个线程请求一个它已经拥有的锁时，请求会成功。这样的锁叫做可重入锁。可重入锁意味着锁是每个线程分配一个，而不是每次调用分配一个。每个锁会有一个 count，每次 lock / unlock 时 +1 / -1。当 count 为零时锁会被释放。

`synchronized` 关键字和 `ReentrantLock` 都是可重入锁。如果 `synchronized` 没有可重入性，当子类重写了父类的一个 synchronized method 话，会导致 deadlock。

### Guarded by

对于一个可变的状态变量，如果每次对该变量的读/写都需要在拥有同一个特定的锁时进行，则称这个变量是 _guarded by_ 这个锁。这样可以保证在同一时刻只有一个线程访问(读/写)这个变量。

### 可中断锁

+ `synchronized` -- 不可中断锁
+ `ReentrantLock` -- 可中断锁

### 公平锁

+ `synchronized` -- 非公平锁
+ `ReentrantLock`, `ReentrantReadWriteLock` -- 可以设置为公平锁或非公平锁

## `synchronized` 关键字

`synchronized` 关键字是一种 locking 机制，是一种 mutex。

_Synchronized methods_:

_Synchronized statements_:

## `java.util.concurrent.locks` 中的锁

+ `Lock`
  + `ReentrantLock`
  + `ReadLock`
  + `WriteLock`
+ `ReadWriteLock`
  + `ReentrantReadWriteLock`

锁相比 `synchronized` 关键字的优点：

+ 可以控制多个线程同时读
+ 可以知道线程是否成功获取了锁

`ReentrantLock`，可重入锁。

## CAS 与无锁数据结构