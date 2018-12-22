# 依赖注入 (Dependency Injection) 与控制反转 (Inversion of Control)

Java DI 规范 (Java Dependency Injection Specification) 中定义的依赖注入，Spring 等框架对其兼容。

+ `@Named` ~= `@Component`
+ `@Inject` ~= `@Autowired`

## Spring

Spring 中的 bean 默认都是 singleton。

### Wire beans

+ Automatic configuration
+ Explicit configuration
    + JavaConfig
    + XML configuration
