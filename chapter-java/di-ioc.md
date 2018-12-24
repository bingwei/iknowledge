# 依赖注入 (Dependency Injection) 与控制反转 (Inversion of Control)

控制反转即“获得依赖对象的过程”被反转了。软件系统中的对象会相互依赖，对象 A 在初始化或运行到某一点的时候，必须拥有一个已创建的对象 B。传统的做法是让 A 控制 B 的创建过程。而使用了 IoC 容器之后，对象 A 将控制权交了出来，由 IoC 容器创建一个 B 对象并注入到 A 需要的地方。对象 A 获得依赖对象的过程，由主动行为变成了被动行为，这称为控制反转。

依赖注入是实现控制反转的方法。依赖注入和控制反转是从不同的角度描述同一件事情：“通过引入 IoC 容器，利用 DI 的方式，实现对象之间的解耦”。

Java DI 规范 (Java Dependency Injection Specification) 中定义的依赖注入，Spring 等框架对其兼容。

+ `@Named` ~= `@Component`
+ `@Inject` ~= `@Autowired`

TODO list:

+ [JavaGuide: Spring](https://github.com/Snailclimb/JavaGuide/blob/master/%E4%B8%BB%E6%B5%81%E6%A1%86%E6%9E%B6/Spring%E5%AD%A6%E4%B9%A0%E4%B8%8E%E9%9D%A2%E8%AF%95.md)
+ DI 和 IoC 的具体定义，以及它们之间的关系
  + 可以参考《软件体系结构》的课件
+ IoC 框架的实现原理是什么？

## Spring

Spring 中的 bean 默认都是 singleton。

### Wire beans

+ Automatic configuration
+ Explicit configuration
  + JavaConfig
  + XML configuration

TODO 学习 Spring MVC 来进一步理解 IoC。

## Spring IoC 原理

基于反射机制：

+ 加载 class
+ 创建对象实例
+ 设置对象 field 的值（原始类型或对象类型）

两篇 Spring IoC 源码阅读（都是 XML-based DI 部分）：

+ [Spring IOC 核心源码学习](https://yikun.github.io/2015/05/29/Spring-IOC%E6%A0%B8%E5%BF%83%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/)
+ [Spring IOC 容器源码分析](https://javadoop.com/post/spring-ioc)

## 参考资料

+ _Spring in Action_