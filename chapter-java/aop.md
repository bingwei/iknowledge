# 面向切面编程 (Aspect-Oriented Programming)

面向切面的常见场景：

+ Logging
+ Security
+ Transaction
+ Caching

如果一个功能分散在程序中的多个位置，这个功能称为 _cross-cutting concerns_。将这些功能与业务逻辑相分离的做法称为面向切面编程 (AOP)。AOP 有助于降低程序的耦合性。

## 术语

+ **Join point** (连接点)
  + Join point 是切面可以切入的地方，切面在 join point 处将特定的代码插入程序正常的执行流中
  + Spring 只支持 method 级别的 join point，而 AspectJ 支持 field 级别的
+ **Aspect** (切面)
  + Aspect 即 advice 加 pointcut
+ **Advice** (通知)
  + 定义了一个切面应当做的工作 (_what_ & _when_)
  + Spring 中的 advice 包括三种类型：
    + Before (`@Before`): 在 method 调用之前工作
    + After* (`@After`, `@AfterReturning`, `@AfterThrowing`): 在 method 调用之后工作
    + Around (`@Around`): wrap 了目标 method，可实现 before 和 after 的所有功能
+ **Pointcut** (切点)
  + 定义了一个切面的作用范围 (_where_)
  + Pointcut 会 match 一些 join point，只有这些 join point 才是切面的作用范围
+ **Introduction** (引入)
  + 一种方案，为已有的类添加新的 method 和 attribute 的方案 
+ **Weave** (织入)
  + 一种动作，将切面应用至目标对象的 join point 的动作
  + Weave 可以发生在编译期、类加载期、运行期，其中 Spring 在运行期进行 weave

## 框架

Spring 的 AOP 框架借鉴了 AspectJ 的 annotation-based 语法。不过 Spring 只支持 method 级别的 AOP（AspectJ 支持 field 级别的 AOP）。

## AOP 实现原理

AOP 一般使用代理模式实现（见设计模式-代理模式）。代理模式的实现方式主要有三种：

+ 静态代理
+ JDK 动态代理：`java.lang.reflect.Proxy`
+ CGLIB 动态代理

| JDK 动态代理 | CGLIB 动态代理 |
| :-: | :-: |
| 基于接口 | 基于类 |
| 生成目标对象接口的子类对象 | 生成目标对象的子类对象 |

CGLIB 代理的能力比 JDK 代理强。Spring 同时支持 JDK 动态代理、AspectJ 动态代理、CGLIB动态代理。Spring 会根据实际的情况选择不同的代理实现，对使用者完全透明。