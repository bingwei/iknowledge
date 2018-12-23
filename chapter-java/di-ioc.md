# 依赖注入 (Dependency Injection) 与控制反转 (Inversion of Control)

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

## 参考资料

+ _Spring in Action_