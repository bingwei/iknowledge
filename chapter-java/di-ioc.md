# 依赖注入 (Dependency Injection) 与控制反转 (Inversion of Control)

Java DI 规范 (Java Dependency Injection Specification) 中定义的依赖注入，Spring 等框架对其兼容。

+ `@Named` ~= `@Component`
+ `@Inject` ~= `@Autowired`

TODO list:

+ DI 和 IoC 的具体定义，以及它们之间的关系
  + 可以参考《软件体系结构》的课件

## Spring

Spring 中的 bean 默认都是 singleton。

### Wire beans

+ Automatic configuration
+ Explicit configuration
  + JavaConfig
  + XML configuration

## 参考资料

+ _Spring in Action_