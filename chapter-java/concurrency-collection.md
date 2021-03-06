# Java Concurrency - 并发集合

## 不可修改的集合

### Unmodifiable view 不可修改的视图

+ `Collection.unmodifiableList()`
+ `Collection.unmodifiableSet()`
+ `Collection.unmodifiableMap()`
+ `Collection.unmodifiableCollection()`

传入一个普通的集合，返回一个 _unmodifiable view_（设计模式-**代理模式**）。相当于一个保护代理，只允许不修改原集合的方法（调用 `add` 等方法会抛出 `UnsupportedOperationException`）。但是如果直接修改原集合，view 集合可以看得到修改结果。

### Immutable collections 不可变集合 (since Java 9)

+ `List.of()`, `List.copyOf()`
+ `Set.of()`, `Set.copyOf()`
+ `Map.of()`, `Map.ofEntries()`, `Map.copyOf()`

返回一个不可变的集合（调用 `add` 等方法会抛出 `UnsupportedOperationException`）。

## Synchronized collections (view) 同步集合(视图)

+ `Collection.synchronizedList()`
+ `Collection.synchronizedSet()`
+ `Collection.synchronizedMap()`
+ `Collection.synchronizedCollection()`

传入一个普通的集合，返回包装后的线程安全的集合（设计模式-**代理模式**）。实际上就是将每个方法 (如 `add`, `get`, `remove`) 包裹在 `synchronized` 块中。（也有例外，没有对 iterator, stream 等方法加锁。）包装过之后，就不要直接操作原来的集合了。

这些集合的缺点是效率较低，而且只能保证**有条件的线程安全**：所有单个操作都是线程安全的，但操作序列不是线程安全的。Client 在进行相互关联的操作的时候仍然必须要加锁（这叫做 *client-side locking*）：

```Java
List list = Collections.synchronizedList(new ArrayList());
    ...
synchronized (list) {
    Iterator i = list.iterator(); // Must be in synchronized block
    while (i.hasNext())
        foo(i.next());
}
```

注意 `Vector`, `Stack`, `Hashtable` 也属于 synchronized collections。但它们早已过时了：

+ `Vector`，并发版本的 `ArrayList`，使用 `CopyOnWriteArrayList` 代替
+ `Stack`，使用 `Deque` 的实现类代替
+ `Hashtable`，并发版本的 `HashMap`，使用 `ConcurrentHashMap` 代替

`StringBuffer` (同步版本的 `StringBuilder`) 也相当于 synchronized collection。它实际上就是在 `StringBuilder` 的方法上加了 `synchronized`。

## Concurrent collections 并发集合 (since Java 5)

为并发而设计的集合，比 synchronized collections 要强大许多。

+ `ConcurrentMap` [interface]
  + `ConcurrentHashMap` (常见)
  + `ConcurrentSkipListMap`
+ `BlockingQueue` [interface]
  + `ArrayBlockingQueue` (常见)
  + `LinkedBlockingQueue` (常见)
  + `PriorityBlockingQueue` (常见)
  + `SynchronousQueue` (常见)
  + `DelayQueue`
  + `BlockingDeque` [interface]
    + `LinkedBlockingDeque`
  + `TransferQueue` [interface]
    + `LinkedTransferQueue`
+ `ConcurrentLinkedQueue`
+ `ConcurrentLinkedDeque`
+ `CopyOnWriteArrayList` (常见)
+ `CopyOnWriteArraySet`

并发替代品：

+ `ArrayList` -> `CopyOnWriteArrayList`
+ `HashMap` -> `ConcurrentHashMap`
+ `TreeMap` -> `ConcurrentSkipListMap`
+ `TreeSet` -> `ConcurrentSkipListSet`
+ `PriorityQueue` -> `PriorityBlockingQueue`

并发集合的 iterator 不会抛出 `ConcurrentModificationException` 异常（参见 Java Collection Framework 章节）。

### ConcurrentMap

+ `ConcurrentHashMap` 是并发版本的 `HashMap`
+ `ConcurrentSkipListMap` 是并发版本的 `TreeMap`

`ConcurrentHashMap` 常用来和 `Hashtable` 比较。`Hashtable` 使用**全表锁**，只能有一个线程同时进行 put/get 操作。`ConcurrentHashMap` 通过细粒度的锁进行优化。

+ Java 7 `ConcurrentMap`: 对每个 segment 加锁
  + `Segment` 继承 `ReentrantLock`，自己本身是锁
  + 每个 segment 包含数组+链表，类似一个 HashMap (1.7)
+ Java 8 `ConcurrentMap`: 对每个桶元素加锁
  + 在 HashMap (1.8) 的基础上，对链表或红黑树的首结点加锁
  + 使用 CAS 和 `synchronized` 加锁

### 阻塞队列： `BlockingQueue`

在 `Queue` 接口的基础上添加一些阻塞行为。如果从空队列中取出元素，或者从满队列中插入元素，会导致等待。`Queue` 接口支持**抛出异常**和**返回特殊值**，而 `BlockingQueue` 接口还额外支持**阻塞**和**超时**。重点在 `put` 和 `take` 两个阻塞方法上。

| 操作 | 抛出异常 | 返回特殊值 | 阻塞 | 超时 |
| :-: | :-: | :-: | :-: | :-: |
| 插入元素 | `add(e)` | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
| 删除元素 | `remove()` | `poll()` | `take()` | `poll(time, unit)` |
| 查看头部元素 | `element()` | `peek()` | N/A | N/A |

三个默认的线程池实现都用到了 blocking queue（两个用 `LinkedBlockingQueue`，一个用 `SynchronousQueue`）。

#### ArrayBlockingQueue 与 LinkedBlockingQueue

都是使用 `ReentrantLock` 和 `Condition` 来实现。

| | ArrayBlockingQueue | LinkedBlockingQueue |
| :-: | :-: | :-: |
| 底层数据结构 | 循环数组 | 带 dummy head 的单链表 |
| 容量限制 | 有限容量 | 默认无限容量，可以设置为有限 |
| 锁的数量 | 一把锁 | 两把锁：putLock 和 takeLock |
| 锁的公平性 | 可以设置公平或非公平 | 非公平 |

#### SynchronousQueue

SynchronousQueue：没有任何容量（容量为 0）的特殊 blocking queue。一个线程 put 的时候必须等待另一个线程 take，反之亦然。它有点类似 CSP 中的 channel 概念。

可以选择公平模式 (FIFO) 或非公平模式。SynchronousQueue 实现了 _dual stack and dual queue_ 算法。对于非公平模式，使用 FIFO 的 dual stack；对于公平模式，使用 LIFO 的 dual queue。

Dual stack 和 dual queue 都是无锁数据结构。底层使用 CAS 和自旋操作 (spin) 实现。

参考：

+ 论文原文：[Nonblocking Concurrent Data Structures with Condition Synchronization](http://www.cs.rochester.edu/research/synchronization/pseudocode/duals.html)

#### PriorityBlockingQueue

就是 PriorityQueue 的线程安全版本，使用基于数组的二叉堆实现。

### 非阻塞队列： `ConcurrentLinkedQueue`, `ConcurrentLinkedDeque`

这两个类内部使用 CAS 来保证线程安全。

### Copy-on-write (COW) 容器：`CopyOnWriteArrayList`, `CopyOnWriteArraySet`

Copy-on-write 将读/写操作区别对待：

+ 写操作加互斥锁，写时先将容器复制一份，写入完成后，再将原容器的引用指向新的容器
+ 读操作不加锁，可以多线程同时读
+ 读/写可以同时进行

Copy-on-write 容器的特点：

+ 适用于读多写少的并发场景
  + 在这种场景下，性能要比 `Vector` 好得多
+ 应当尽量批量进行写操作，防止过多的性能开销和内存占用
+ 使用 iterator 进行读时，读的是当前容器的 snapshot
  + Iterator 创建之后，对容器进行的修改不会反映在 iterator 中
  + Iterator 不会抛出 `ConcurrentModificationException`

`CopyOnWriteArraySet` 由于使用数组实现（实际上是基于 `CopyOnWriteArrayList` 实现的），查找等操作的时间复杂度是 O(n)，所以适用于小集合的场景。

## 参考

+ [并发集合类 - ConcurrentHashMap 和 CopyOnWriteArrayList 提供线程安全性和已改进的可伸缩性](https://www.ibm.com/developerworks/cn/java/j-jtp07233/index.html)
+ [JDK 中的并发集合](http://novoland.github.io/%E5%B9%B6%E5%8F%91/2014/07/26/%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88.html)
+ [解读 Java 并发队列 BlockingQueue](https://javadoop.com/post/java-concurrent-queue#%E6%80%BB%E7%BB%93)