# 语法

## transient 关键字

## volatile 关键字

# 序列化

[Java 序列化的高级认识](https://www.ibm.com/developerworks/cn/java/j-lo-serial/)

Java序列化规则总结: [StackOverflow](https://stackoverflow.com/questions/16442802/will-serialization-save-the-superclass-fields/16442977#16442977)

_Effective Java_ 第11章中有一些关于序列化的高级话题。

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

## Lambda表达式与函数式接口

Lambda类似匿名内部类的语法糖，但JVM层面，两者有显著不同。

举Java Tutorial中的筛选的例子，类似Comparator，是一种策略模式。

传递“行为”（对比传递“值”）--> 策略模式 --> 匿名内部类 --> lambda表达式

哪里可以用? --> 一切函数式接口
