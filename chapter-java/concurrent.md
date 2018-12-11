### volatile

`volatile` 关键字是关于 visibility 的。

### Synchronized collections

_client-side locking_

### Unmodifiable view

+ `Collection.unmodifiableList()`
+ `Collection.unmodifiableMap()`
+ `Collection.unmodifiableSet()`

返回的都是一个 _unmodifiable view_。也就是说返回的对象是不可以修改的，但是对原对象的修改，view 对象可以看得到修改结果。

### Copy-on-write (COW) 容器

+ `CopyOnWriteArrayList`
+ `CopyOnWriteArraySet`