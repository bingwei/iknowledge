# Java 集合框架

Java 集合框架 (_Java Collections Framework_, JCF) 始于 Java 1.2。

## Collection hierarchy 类结构

集合框架分为两大部分：

+ `Collection` 及其子类
+ `Map` 及其子类
  + 并不是真正的集合，但可以像集合一样操作

所有接口：

![JCF interfaces](collection/collection-interfaces.png)

TODO: `Vector`, `Stack`, `Hashtable`, `Enumeration`

类结构（来自 _Thinking in Java_ 第 17 章）：

![JCF classes](collection/collection-classes.jpg)

## 实现原理

接口与对应的实现：

| 接口 | Hash Table | Resizable Array | Balanced Tree | Linked List | Hash Table + Linked List |
| :-: | :-: | :-: | :-: | :-: | :-: |
| `List` | | `ArrayList` | | `LinkedList` | |
| `Deque` | | `ArrayDeque` | | `LinkedList` | |
| `Set` | `HashSet` | | `TreeSet` | | `LinkedHashSet` |
| `Map` | `HashMap` | | `TreeMap` | | `LinkedHashMap` |

注意没有 `LinkedDeque` 类，只有 `LinkedList`。`LinkedList` 既实现了 `List` 接口，也实现了 `Deque` 接口。

几个 `Set` 的实现都是利用了对应的 `Map` 类，只是包装的接口（设计模式-**适配器模式**）：

+ `TreeSet` -> `TreeMap`
+ `HashSet` -> `HashMap`
+ `LinkedHashSet` -> `LinkedHashMap`

`TreeMap` 底层使用红黑树实现。https://zhuanlan.zhihu.com/p/24795143?refer=dreawer

AbstractCollection, AbstractSet, AbstractList, AbstractSequentialList and AbstractMap 提供了一些基本的实现。

TODO: 几个关键类的源码剖析

集合类如果不支持某个操作，会抛出 `UnsupportedOperationException`。

## 迭代器

TODO: 迭代器，fail-fast, ConcurrentModificationException

## 其他

哪些集合接受 null value？

+ 接受
  + `ArrayList`
  + `LinkedList`
  + `CopyOnWriteArrayList`
  + `HashSet`?, `LinkedHashSet`?
+ 不接受
  + `TreeSet`?
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