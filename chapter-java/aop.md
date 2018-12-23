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

## 框架

Spring 的 AOP 框架借鉴了 AspectJ 的 annotation-based 语法。不过 Spring 只支持 method 级别的 AOP（AspectJ 支持 field 级别的 AOP）。

## AOP 实现原理

使用代理模式实现 AOP（具体见设计模式章节中的对应内容）：

+ JDK 动态代理 `java.lang.reflect.Proxy`
+ CGLIB 动态生成代理类

| JDK 动态代理 | CGLIB 动态代理 |
| :-: | :-: |
| 基于接口 | 基于类 |
| 生成目标对象接口的子类对象 | 生成目标对象的子类对象 |

CGLIB 代理的能力比 JDK 代理强。Spring 同时使用了这两种实现方式，当目标类有接口的时候，使用 JDK 代理；否则使用 CGLIB。