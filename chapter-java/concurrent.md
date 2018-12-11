# Java Concurrent

## volatile

`volatile` 关键字是关于 visibility 的。

## Concurrent constructs

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
+ `ConcurrentLinkedQueue`
+ `CopyOnWriteArrayList`
+ `CopyOnWriteArraySet`

### Copy-on-write (COW) 容器

+ `CopyOnWriteArrayList`
+ `CopyOnWriteArraySet`