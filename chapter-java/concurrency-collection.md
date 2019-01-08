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

TODO：这些集合有什么缺点

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

并发集合的 iterator 不会抛出 `ConcurrentModificationException` 异常。

TODO：ConcurrentModificationException

### ConcurrentMap

+ `ConcurrentHashMap` 是并发版本的 `HashMap`
+ `ConcurrentSkipListMap` 是并发版本的 `TreeMap`

`ConcurrentHashMap` 常用来和 `Hashtable` 比较。`ConcurrentHashMap` 使用**分段锁**，性能要比 `Hashtable`（使用**全表锁**）好很多。

### 阻塞队列： `BlockingQueue`

在 `Queue` 接口的基础上添加一些阻塞行为。如果从空队列中取出元素，或者从满队列中插入元素，会导致等待。`Queue` 接口支持**抛出异常**和**返回特殊值**，而 `BlockingQueue` 接口还额外支持**阻塞**和**超时**。重点在 `put` 和 `take` 两个阻塞方法上。

| 操作 | 抛出异常 | 返回特殊值 | 阻塞 | 超时 |
| :-: | :-: | :-: | :-: | :-: |
| 插入元素 | `add(e)` | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
| 删除元素 | `remove()` | `poll()` | `take()` | `poll(time, unit)` |
| 查看头部元素 | `element()` | `peek()` | N/A | N/A |

三个默认的线程池实现都用到了 blocking queue（两个用 `LinkedBlockingQueue`，一个用 `SynchronousQueue`。

ArrayBlockingQueue：数组实现，有限容量，`ReentrantLock` 控制。

LinkedBlockingQueue：链表实现，有限/无限容量，`ReentrantLock` 控制。

SynchronousQueue：容量为 0，可以选择公平模式 (FIFO) 或非公平模式。可以用来保证公平性。

PriorityBlockingQueue 就是 PriorityQueue 的线程安全版本，使用基于数组的二叉堆实现。

### 非阻塞队列： `ConcurrentLinkedQueue`, `ConcurrentLinkedDeque`

这两个类内部使用 CAS 来保证线程安全。

### Copy-on-write (COW) 容器

+ `CopyOnWriteArrayList`
+ `CopyOnWriteArraySet`

Copy-on-write 将读/写操作区别对待：

+ 写操作加互斥锁，写时先将容器复制一份，写入完成后，再将原容器的引用指向新的容器
+ 读操作不加锁，可以多线程同时读
+ 读/写可以同时进行

Copy-on-write 容器的特点：

+ 适用于读多写少的并发场景
  + 在这种场景下，性能要比 `Vector` 好得多
+ 适合批量进行写操作，放置过多的复制带来的性能开销和内存占用
+ 使用 iterator 进行读时，读的是当前容器的 snapshot
  + Iterator 创建之后，对容器进行的修改不会反映在 iterator 中
  + Iterator 不会抛出 `ConcurrentModificationException`

`CopyOnWriteArraySet` 由于使用数组实现（实际上是基于 `CopyOnWriteArrayList` 实现的），查找等操作的时间复杂度是 O(n)，所以适用于小集合的场景。

## 并发集合实现原理

### `ConcurrentHashMap`

### `ArrayBlockingQueue`

### `LinkedBlockingQueue`

### `PriorityBlockingQueue`

### `SynchronousQueue`

### `CopyOnWriteArrayList`

## 参考

+ [并发集合类 - ConcurrentHashMap 和 CopyOnWriteArrayList 提供线程安全性和已改进的可伸缩性](https://www.ibm.com/developerworks/cn/java/j-jtp07233/index.html)
+ [JDK 中的并发集合](http://novoland.github.io/%E5%B9%B6%E5%8F%91/2014/07/26/%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88.html)
+ [解读 Java 并发队列 BlockingQueue](https://javadoop.com/post/java-concurrent-queue#%E6%80%BB%E7%BB%93)