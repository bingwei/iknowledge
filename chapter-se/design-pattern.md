# 设计模式

如何学习设计模式：要能讲解出来，能举出例子。

分类（以 GoF 书籍为准）：

+ 创建型模式 (creational patterns)
  + Singleton 单例
  + Builder 构建
  + Prototype 原型
  + Factory method 工厂方法
  + Abstract factory 抽象工厂
+ 行为型模式 (behavioral patterns)
  + Strategy 策略
  + State 状态
  + Observer 观察者
  + Command 命令
  + Iterator 迭代器
  + Mediator 中介
  + Interpreter 解释器
  + Template method 模板方法
  + Chain of responsibility 责任链
  + Memento 备忘录
  + Visitor 访问者
+ 结构型模式 (structural patterns)
  + Decorator 装饰者
  + Facade 外观
  + Adapter 适配器
  + Proxy 代理
  + Bridge 桥接
  + Flyweight 享元（蝇量）
  + Composite 组合

## 创建型模式

### Singleton 单例

TODO

### Builder 构建

TODO

### Prototype 原型

TODO

### Factory method 工厂方法

TODO

### Abstract factory 抽象工厂

TODO

## 行为型模式 (behavioral patterns)

### Strategy 策略

_Head First Design Patterns_ 中鸭子游戏的例子：

困难：
如果使用继承来复用代码，会给一些子类添加上不必要的行为（如橡皮鸭不会飞）。
如果使用Flyable接口，会造成代码无法复用。

方法：
由于鸭子的行为（飞行行为，叫声行为）是多变的，为了分开变化和不变化的部分，我们将鸭子的两个行为取出来作为新的类。
这样，鸭子类不需要知道行为的实现细节，可以针对接口进行编程。

好处：
这些行为可以相互替换，也可以加入新的行为。

+ `java.io.FileFilter`与`java.io.FilenameFilter`，传入`java.io.File.listFiles()`
+ `java.util.Comparator`，传入`java.util.Arrays.sort()`（另有`java.util.List.sort()`与`java.util.Collections.sort()`，与`Arrays.sort()`实际相同）
+ `android.support.v7.widget.RecyclerView.LayoutManager`，有子类`LinearLayoutManager`, `GridLayoutManager`；用户可自定义子类，并通过`RecyclerView.setLayoutManager()`动态设置

### State 状态

API 中没有找到好的例子，但实际系统中应该经常能用到。

### Observer 观察者

Java 标准库中有 `java.util.Observable` 类和 `java.util.Observer` 类。但 `Observable` 类有明显的缺点，一般没有人用。可以使用 Guava 中的 `EventBus`。

### Command 命令

+ `java.lang.Runnable`与`java.util.concurrent.Callable`，交给`ExecutorService`执行
+ 数据库记录日志，用来undo, redo

### Iterator 迭代器

+ `java.util.Iterator` 与 `java.util.ListIterator`

### Mediator 中介

TODO

### Interpreter 解释器

TODO

### Template method 模板方法

+ `java.io.InputStream`, `java.io.Reader` 中的 `read()` 等； `java.io.OutputStream`, `java.io.Writer` 中的 `write()` 等
+ `java.util.AbstractList` 中的 `add()`, `get()`, `remove()` 等；`java.util.AbstractMap` 中的 `put()`, `entrySet()`等
+ `android.app.Activity` 中的 `onCreate()`, `onResume()`, `onPause()` 等，提供默认行为，子类可扩充
+ `javax.servlet.http.HttpServlet` 中的 `doGet()`, `doPost()` 等，默认返回405错误，需要子类覆盖实现

### Chain of responsibility 责任链

+ 很多 logger，如`java.util.logging.Logger`（有待验证）
+ `okhttp3.Interceptor` （[OkHttp 的 Interceptors 与责任链模式](http://nettee.github.io/posts/2018/OkHttp-Interceptors-and-Chain-of-Responsibility-Pattern/))
+ Android 中对 TouchEvent 的处理（思路接近）

### Memento 备忘录

TODO

### Visitor 访问者

TODO

## 结构型模式 (structural patterns)

### Decorator 装饰者

Java I/O 标准库使用装饰器模式的好处：[stackoverflow](https://stackoverflow.com/questions/6366385/use-cases-and-examples-of-gof-decorator-pattern-for-io)

### Facade 外观

TODO

### Adapter 适配器

TODO

### Proxy 代理

_Head First Design Patterns_ 中给出了两个很好的例子：

+ **远程代理**: Java RMI
+ **虚拟代理**: 对于一个创建开销大的对象（如需要从网上下载专辑封面的对象），当对象在创建前或创建中时，虚拟代理处理请求；当对象创建完成后，虚拟代理会直接将请求委托给真实对象

`java.lang.reflect` 包可以支持动态代理。AOP 可以使用动态代理来实现。

### Bridge 桥接

TODO

### Flyweight 享元（蝇量）


Java 中的例子：[Design Patterns in the Real World: Flyweight](https://tamasgyorfi.net/2016/05/30/design-patterns-in-the-real-world-flyweight/)


### Composite 组合

TODO

## 设计模式间的区分

+ Stategy vs. State: 见 _Head First Design Patterns_ 第10章
+ Chain of Responsibility vs. Decorator: [Why would I ever use a Chain of Responsibility over a Decorator?](https://stackoverflow.com/questions/747913/why-would-i-ever-use-a-chain-of-responsibility-over-a-decorator)
+ [那些相似的设计模式的区别](https://blog.csdn.net/jinzhuojun/article/details/11555595)

## 参考资料

+ [Wikipedia: Software design pattern](https://en.wikipedia.org/wiki/Software_design_pattern)
+ [图说设计模式](https://design-patterns.readthedocs.io/zh_CN/latest/index.html)

设计模式的对应例子：

+ Java 标准库里的设计模式
  +  [Examples Design Patterns in Java's core libraries](https://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns-in-javas-core-libraries).
  + [Patterns in Java APIs](http://cecs.wright.edu/~tkprasad/courses/ceg860/paper/node26.html)
+ Android 库里的设计模式
  + [Android 源码设计模式分析](https://github.com/simple-android-framework/android_design_patterns_analysis)
