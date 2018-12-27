# 设计模式 - 结构型模式

+ 结构型模式 (structural patterns)
  + Decorator 装饰者
  + Adapter 适配器
  + Facade 外观
  + Proxy 代理
  + Bridge 桥接
  + Flyweight 享元（蝇量）
  + Composite 组合

## Decorator 装饰者

### 目的

装饰者模式是为了在**不改变接口**的情况下，加入新的功能。

### 例子

Java I/O 标准库是最好的例子，`InputStream`, `OutputStream`, `Reader`, `Writer` 的子类

### 参考

Java I/O 标准库使用装饰器模式的好处：[stackoverflow](https://stackoverflow.com/questions/6366385/use-cases-and-examples-of-gof-decorator-pattern-for-io)

## Adapter 适配器

### 目的

适配器将一个接口**转换**成（用户期望的）另一个接口。适配器让原本接口不兼容的类可以合作。就像真实世界的适配器：让美式插头能插在欧式插座上。

### 例子

+ 兼容暴露 `Enumeration` 接口的老旧 Java 代码：
  + `Enumeration` -> `Iterator`
+ Java I/O: 
  + `InputStreamReader` (`InputStream` -> `Reader`)
  + `OutputStreamReader` (`OutputStream` -> `Writer`)

适配器模式可以通过多重继承来实现（_Head First Design Patterns_ 第 7 章）。

### 参考

+ _Head First Design Patterns_ 第 7 章

## Facade 外观

### 目的

外观模式是为了让接口更简单。外观模式在一个子系统的一群（复杂的）接口上**包装**出简单的接口，使其容易使用。外观模式不会“封装”原有的接口，需要的人仍可以使用系统完整的功能。

### 例子

_Head First Design Patterns_ 第 7 章中的家庭影院的例子：用一个接口包装所有设备的操作。

### 参考

+ _Head First Design Patterns_ 第 7 章

## Proxy 代理

### 典型特征

对象A（代理对象）将功能委托给对象B（被代理对象），两个对象**接口相同**

### 例子

_Head First Design Patterns_ 中给出了两个很好的例子：

+ **远程代理**: Java RMI
+ **虚拟代理**: 对于一个创建开销大的对象（如需要从网上下载专辑封面的对象），当对象在创建前或创建中时，虚拟代理处理请求；当对象创建完成后，虚拟代理会直接将请求委托给真实对象

`java.lang.reflect.Proxy` 可以创建动态代理，在运行时实现代理模式。

### 用途

Spring AOP 基于动态代理实现。

## Bridge 桥接

### 目的

将*抽象*（一个类）和*实现*（一个类做的事情）解耦，使得两者可以独立地变化，而不会相互影响。通俗地说，桥接模式**连接了两个维度**。否则，两个维度的变化将导致组合爆炸。

桥接适合使用在需要使用抽象维度的系统上，如笔-颜色、电视机-遥控器、手机品牌-手机功能、汽车品牌-手动挡自动挡。

## 例子

https://stackoverflow.com/questions/319728/when-do-you-use-the-bridge-pattern-how-is-it-different-from-adapter-pattern

JDBC 是吗？https://stackoverflow.com/questions/46228420/why-is-jdbc-is-a-typical-example-of-bridge-design-pattern

### 参考

+ _Head First Design Patterns_ 附录

## Flyweight 享元（蝇量）

### 目的

节省内存

### 典型特征

创建对象时，返回一个缓存的实例

### 例子

+ JVM 的 integer cache
  + -128 ~ 127 （上限可设置）的整数对象会预先缓存，不会生成重复的对象
+ `String.intern()`
  + 返回 string pool 中的一份缓存

参考：[Design Patterns in the Real World: Flyweight](https://tamasgyorfi.net/2016/05/30/design-patterns-in-the-real-world-flyweight/)

## Composite 组合

TODO