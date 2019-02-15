# Java 集合框架

Java 集合框架 (_Java Collections Framework_, JCF) 始于 Java 1.2。

## Collection hierarchy 类结构

集合框架分为两大部分：

+ `Collection` 及其子类
+ `Map` 及其子类
  + 并不是真正的集合，但可以像集合一样操作

所有接口：

![JCF interfaces](collection/collection-interfaces.png)

### List

### Set

### Queue / Deque

### Map

AbstractCollection, AbstractSet, AbstractList, AbstractSequentialList and AbstractMap 提供了一些基本的实现。

## 实现原理

TODO: 几个关键类的源码剖析

接口与对应的实现：

| 接口 | Hash Table | Resizable Array | Balanced Tree | Linked List | Hash Table + Linked List |
| :-: | :-: | :-: | :-: | :-: | :-: |
| `List` | | `ArrayList` | | `LinkedList` | |
| `Deque` | | `ArrayDeque` | | `LinkedList` | |
| `Set` | `HashSet` | | `TreeSet` | | `LinkedHashSet` |
| `Map` | `HashMap` | | `TreeMap` | | `LinkedHashMap` |

`TreeSet`, `HashSet`, `LinkedHashSet` 都是基于对应的 `Map` 类实现，只是包装了接口（设计模式-**适配器模式**）。

### 动态数组：`ArrayList`, `ArrayDeque`

可增长的动态数组

+ 支持随机访问，`get(i)` = O(1)
+ 插入/删除需要拷贝数据

### 链表：`LinkedList`

双向链表

+ 不支持随机访问，`get(i)` = O(n)
+ 插入/删除只需移动指针，不需要拷贝数据

注意没有 `LinkedDeque` 类，只有 `LinkedList`。`LinkedList` 既实现了 `List` 接口，也实现了 `Deque` 接口。

+ Java 6 之前为双向循环链表
  + 使用一个 dummy entry，`header` 指向 dummy entry
  + 在尾部插入实际上就是在 `header` 的前面插入
+ Java 7 去掉了循环
  + 使用 `first`, `last` 分别指向链表头部和尾部


### 散列表：`HashMap`

散列表的实现原理：

+ 计算 hash
  + 调用 `key.hashCode()` 得到 hash code
  + 使用扰动函数处理 hash code，得到最终的 hash 值
    + 扰动函数可以避免 `hashCode()` 方法实现得太差，导致太多 hash 冲突
+ 寻找 bucket
  + Bucket 的数量设置为 2 的幂（默认初始值 16）
    + 取模操作转化为位操作，计算方便
  + 对比：`Hashtable` 的 bucket 数量为质数
+ 处理 hash 冲突
  + Java 7 以前，直接使用（单？）链表
  + Java 8 以后，当链表长度超过阈值 (8)，链表会转化成**红黑树**
+ 扩容 (resize)
  + 扩容阈值 threshold = bucket 数量 * load factor
  + 当 size > threshold 时扩容
  + Load factor 默认为 0.75
  + Java 8 的新扰动函数，使得扩容时不需要重新计算 hash

### 平衡树：`TreeMap`

`TreeMap` 底层使用红黑树实现。https://zhuanlan.zhihu.com/p/24795143?refer=dreawer

使用 `TreeMap` 时，要么在构造时传入 comparator，要么每个 key 实现 `Comparable` 接口，否则会抛出 `ClassCastException`。

### 散列表 + 链表：`LinkedHashMap`

`LinkedHashMap` 基于 `HashMap` 进行扩展，使用双向链表保持顺序性。可以选择两种排序方式：

+ 根据写入顺序排序
+ 根据访问顺序排序

### 其他

TODO: `Vector`, `Stack`, `Hashtable`, `Enumeration`

`ArrayList` 实现了 `RandomAccess` 接口。`RandomAccess` 是空接口，只是起到标识作用。在 `binarySearch()` 方法中，如果传入的 list 实现了 `RandomAccess` 接口，则调用 `indexedBinarySearch()`，否则调用 `iteratorBinarySearch()` 方法。

集合类如果不支持某个操作，会抛出 `UnsupportedOperationException`。

## 迭代器

### Concurrent modification & fail-fast iterator

如果一边操作集合的迭代器，另一边在修改集合，这时候迭代器的行为是不确定的，可能产生 undefined 的结果。一些迭代器在运行时会进行检查，如果发现集合被修改，会抛出 `ConcurrentModificationException`。这种迭代器称为 _fail-fast iterator_。

`AbstractList` 中定义了 `modCount` 变量，表示 list 被修改的次数。`iterator()` 或 `listIterator()` 返回的迭代器中，每次调用 `previous()`, `next()`, `add()`, `set()`, `remove()` 时都会检查这个变量。如果它发生了预料之外的变化，则迭代器抛出 `ConcurrentModificationException`。注意，迭代器的 `remove()` 等方法会造成 `modCount` 改变，但这是预料之中的变化。

+ 无论是多线程环境还是单线程环境，都可能出现 concurrent modification
+ 迭代器与 for-each 循环等价，因此在 for-each 循环中修改集合也会出现 concurrent modification
+ `Vector` 或 `Collection.synchronizedList()` 的迭代器也会产生这个问题
  + 因为它们并非绝对线程安全
  + 使用迭代器时必须加锁
+ `CopyOnWriteArrayList` 可以解决 concurrent modification 的问题
  + 迭代器反映当前容器的 snapshot

## 对容器元素的要求

### 比较相等： `equals()` 与 `hashCode()`

+ 覆盖 `equals()` 表达“值相等”的含义
  + `equals()` 应当满足自反性、对称性、传递性
  + `equals()` 的参数类型必须是 `Object`，这样才能保证重写 `Object.equals()`
+ 覆盖 `equals()` 时同时也要覆盖 `hashCode()`
  + Java 约定：相等的对象必须有相等的 hash code
  + `HashMap`, `HashSet`, `Hashtable` 的 key 使用 hash code 进行散列
  + 如果只覆盖 `equals()`，而不覆盖 `hashCode()`，`HashMap` 等无法正常工作

自己的例子：ASM 返回的 class descriptor，使用 `ClassName` 类来表示，内部根据 '$' 分离出 class name 和 inner names。又需要使用 `ClassName` 作为 key 来去重，得到不重复的 class names。于是 `ClassName` 覆盖了 `hashCode()`。

参考：_Effective Java_ #8, #9

### 比较大小： `compareTo()` 与 `Comparator`

+ 覆盖 `compareTo()` 表达“值大小”的含义
  + `compareTo()` 应当满足反对称性、传递性
  + 使用 <, <=，>, >=, = 来比较整数基本类型
  + 使用 `Float.compare(f1, f2)` 和 `Double.compare(d1, d2)` 来比较浮点数
+ `compareTo()` 的关系应当与 `equals()` 的关系一致
  + `TreeMap`, `TreeSet` 等的 key 使用 `compareTo()` 进行比较（如果没有外部 `Comparator`）
  + 如果不遵守，`TreeMap` 工作时就无法遵守 `Map` 的（基于 `equals` 的）约定
  + 反例：`BigDecimal("1.0")` 和 `BigDecimal("1.00")`
    + `equals()` 返回 false，而 `compareTo()` 返回 0
    + 将它们添加到 `HashMap` 中（使用 `equals` / `hashCode`），size = 2
    + 将它们添加到 `TreeMap` 中（使用 `compareTo`），size = 1

参考：_Effective Java_ #12

## 其他

哪些集合接受 null value？

+ 接受
  + `ArrayList`
  + `LinkedList`
  + `CopyOnWriteArrayList`
  + `HashMap` (key 和 value 皆可)
  + `HashSet`
+ 不接受
  + `TreeSet`?
  + `ConcurrentHashMap`
  + `ConcurrentSkipListSet`
  + `ArrayDeque`

## Java 集合框架发展历史

+ Java 1.2
  + 引入集合框架
+ Java 5
  + 增强的 for 循环（利用 iterator）
+ Java 6
  + 引入 `Deque`，取代 `Stack`

## 参考资料

+ [Collections framework overview](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)
+ [深入理解 Java 集合框架](https://github.com/CarpenterLee/JCFInternals)
+ JCF 类结构（来自 _Thinking in Java_ 第 17 章）：

![JCF classes](collection/collection-classes.jpg)