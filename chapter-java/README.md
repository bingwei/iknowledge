# Java

## Roadmap

+ 基础知识
  + _Core Java Volume I - Fundamentals_
+ Spring 框架
  + _Spring in Action_
  + _Spring Boot in Action_
  + 《精通Spring 4.x》（工具书）
+ 进阶知识
  + _Effective Java_
  + Google Guava
+ 并发
  + _Java Concurrency in Action_
+ 性能与底层细节
  + _Java Performance: The Definitive Guide_
  + 《深入理解Java虚拟机》周志明
+ 编程思想
  + _Thinking in Java_ （宏观角度思考）
+ 设计模式

## 语言机制

### 语法

+ transient 关键字
+ volatile 关键字
+ strictfp 关键字

TODO 字符串 intern

### Checked exception

Java 中异常类的继承关系：

+ `Throwable`
  + `Error`
  + `Exception`
    + `RuntimeException`
    + ...

其中 `Error` 不算 exception（表示无法恢复的错误），而 `Exception` 则分为 checked / unchecked：

+ _Checked exception_
  + `Exception` 的子类（但不是 `RuntimeException` 的子类）
  + 要么需要 catch，要么需要在 method 中声明 `throws ...`
+ _Unchecked exception_
  + `RuntimeException` 的子类

Checked exception 似乎是 Java 的特色。C++ 中只有 unchecked exception。Java 的 checked exception 其实是一个很不错的设计，但是由于各种库的滥用，实际使用时非常糟糕：

+ throws 声明会不断向上传播
+ throws 声明也是方法签名（？）的一部分，override 的时候必须 throws 同样的异常
  + TODO method signature

参考：[Java设计出checked exception有必要吗？ - 知乎](https://www.zhihu.com/question/30428214)

### 函数式编程

[Java 8 函数式编程](functional.md)

### 序列化

[Java 序列化的高级认识](https://www.ibm.com/developerworks/cn/java/j-lo-serial/)

Java序列化规则总结: [StackOverflow](https://stackoverflow.com/questions/16442802/will-serialization-save-the-superclass-fields/16442977#16442977)

_Effective Java_ 第11章中有一些关于序列化的高级话题。

### 反射/自省 (Reflection)

Java 反射的基本内容：[深入解析 Java 反射 - 基础](https://www.sczyh30.com/posts/Java/java-reflection-1/)

TODO: 从 JVM 层面理解反射

TODO: JDK 动态代理 `java.lang.reflect.Proxy`；从 JVM 层面理解 JDK 动态代理

### Native method

[java native 方法及 JNI 实例](https://blog.csdn.net/xw13106209/article/details/6989415)

## 标准库

+ [Collection](collection.md)
+ [I/O](io.md)
+ [Concurrency](concurrency.md)

## 模式与框架

+ Spring
  + [依赖注入 (DI) / 控制反转 (IoC)](di-ioc.md)
  + [面向切面编程 (AOP)](aop.md)
+ RxJava
  + TODO [RxJava 从入门到放弃再到不离不弃](https://www.daidingkang.cc/2017/05/19/Rxjava/)
