# 面向切面编程 (Aspect-Oriented Programming)

面向切面的常见场景：

+ Logging
+ Security
+ Transaction
+ Caching

如果一个功能分散在程序中的多个位置，这个功能称为 _cross-cutting concerns_。将这些功能与业务逻辑相分离的做法称为面向切面编程 (AOP)。AOP 有助于降低程序的耦合性。

## 术语

+ **Advice**
+ **Pointcut**
+ **Join point**

Spring 只支持 method 级别的 AOP，AspectJ 支持更细粒度的。