# 函数式编程

参考文章：
+ [Java Tutorials: Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
+ [JDK 8 函数式编程入门](https://www.cnblogs.com/snowInPluto/p/5981400.html)
+ [《深入理解 Java 函数式编程》系列文章](http://www.cnblogs.com/CarpenterLee/p/6729368.html)
+ [Java 8 函数式编程](https://leongfeng.github.io/2016/11/18/java8-function-program-learning/)


1. lambda表达式
2. Stream介绍
4. Stream基本用法(map, filter, reduce), 与内置函数式接口
4. Stream collector
5. Stream 源码剖析

## Lambda表达式与函数式接口

Lambda类似匿名内部类的语法糖，但JVM层面，两者有显著不同。

举Java Tutorial中的筛选的例子，类似Comparator，是一种策略模式。

传递“行为”（对比传递“值”）--> 策略模式 --> 匿名内部类 --> lambda表达式

哪里可以用? --> 一切函数式接口

## Stream 源码剖析

[这篇文章](http://www.cnblogs.com/CarpenterLee/archive/2017/03/28/6637118.html)写得不错。

流水线(Pipeline)，由若干个stage组成。其中`Head`表示source stage，`StatelessOp`/`StatefulOp`表示 intermediate stage。

疑问：为何pipeline的子类是stage？

`Sink`类是核心数据结构，其中包括四个核心方法：`begin()`, `accept()`, `end()`, `cancellationRequested()`。

+ 有状态操作(stateful op)需要begin/end 配合accept进行
+ 短路操作需要cancellationRequested配合accept进行
