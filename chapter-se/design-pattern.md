# 设计模式

如何学习设计模式：要能讲解出来，能举出例子。

分类（以 GoF 书籍为准）：

+ 创建型模式 (creational patterns)
  + Singleton 单例
  + Builder 构建（生成器）
  + Prototype 原型
  + Factory method 工厂方法
  + Abstract factory 抽象工厂
+ 结构型模式 (structural patterns)
  + Decorator 装饰者
  + Adapter 适配器
  + Facade 外观
  + Proxy 代理
  + Bridge 桥接
  + Flyweight 享元（蝇量）
  + Composite 组合
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

## Singleton Pattern 单例模式

TODO 利用 Java `<clinit>` 的单例
TODO 两次加锁的单例

### 写法

Eager initialization:

```Java
public class Singleton {
    // 在类加载的时候就实例化，类加载较慢
    private static Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {
        return instance;
    }
}
```

Lazy initialization (Double checked locking):

```Java
public class Singleton {
    // volatile 关键字是必须的，也是这个关键字的典型使用场景
    private volatile static Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        // 第一次 check，避免不必要的同步
        if (instance == null) {
            synchronized (Singleton.class) {
                // 第二次 check，避免重复实例化
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

Class loader:

```Java
public class Singleton { 
    private Singleton() {}
    public static Singleton getInstance(){  
        return SingletonHolder.instance;  
    }  
    private static class SingletonHolder {  
        private static Singleton instance = new Singleton();  
    }  
}
```

Enum:

```Java
public enum Singleton {
    INSTANCE;
}
```

### 参考

+ _Effective Java_, #71
+ [如何正确地写出单例模式](http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/)

## Builder Pattern 建造者模式

### 目的

封装对象的创建过程。

### 例子

Java 的 `StringBuilder`，从名字就可以记住。

很多 Java 类库提供的链式构造方法也是建造者模式的一种形式。_Effective Java_ #2 中讨论了链式构造方法。如 OkHttp:

```Java
Request request = new Request.Builder()  
    .url(url)
    .header("Accept", "text/html")
    .build();
```

## Prototype Pattern 原型模式

通过复制一个已经存在的实例（*原型*）来返回新的实例, 而不是新建实例。

好处：

+ 可以方便地实例化一个**在运行时刻指定**的类
+ 可以不用写复杂的工厂类

复制可以通过 Java 中的 `Clonable` 实现。

## 三种工厂模式

### (Simple) Factory 简单工厂

+ 目的：封装对象创建的过程
+ 方法：通过工厂类创建对象（工厂类只有一个，没有子类）
+ 产品：创建的对象有相同的接口，client 通过接口引用对象

### Factory Method Pattern 工厂方法模式

+ 目的：封装对象创建的过程
+ 方法：定义抽象工厂类，包含**工厂方法**，让子类决定实例化哪一个类
+ 产品：创建的对象有相同的接口，client 通过接口引用对象

### Abstract Factory Pattern 抽象工厂模式

+ 目的：封装**对象家族**创建的过程
+ 方法：定义抽象工厂类，包含多个创建不同对象的接口，让子类决定实例化哪一个类
+ 产品：创建的对象家族有相同的**接口家族**，client 通过接口家族引用对象

特点：

+ 抽象工厂模式中的每个方法都像是一个工厂方法模式
+ 抽象工厂模式可以将一系列相关的产品集合起来

### 参考

+ _Head First Design Patterns_ 第 4 章
+ [Design Patterns: Factory vs Factory method vs Abstract Factory - StackOverflow](https://stackoverflow.com/questions/13029261/design-patterns-factory-vs-factory-method-vs-abstract-factory)

## Decorator Pattern 装饰者模式

### 目的

装饰者模式是为了在**不改变接口**的情况下，加入新的功能。装饰者模式比类继承更加灵活。

### 例子

Java I/O 标准库是最好的例子，`InputStream`, `OutputStream`, `Reader`, `Writer` 的子类

### 参考

Java I/O 标准库使用装饰器模式的好处：[stackoverflow](https://stackoverflow.com/questions/6366385/use-cases-and-examples-of-gof-decorator-pattern-for-io)

## Adapter Pattern 适配器模式

### 目的

适配器将一个接口**转换**成（用户期望的）另一个接口。适配器让原本接口不兼容的类可以合作。就像真实世界的适配器：让美式插头能插在欧式插座上。

### 例子

+ 兼容老旧 Java 代码中使用的 `Enumeration`：
  + `Enumeration` -> `Iterator`
  + `Iterator` -> `Enumeration`
+ Java I/O: 
  + `InputStreamReader` (`InputStream` -> `Reader`)
  + `OutputStreamReader` (`OutputStream` -> `Writer`)

适配器模式可以通过多重继承来实现（_Head First Design Patterns_ 第 7 章）。多重继承还能用来实现双向适配器。

### 参考

+ _Head First Design Patterns_ 第 7 章

## Facade Pattern 外观模式

### 目的

外观模式是为了让接口更简单。外观模式在一个子系统的一组（复杂的）接口上**包装**出简单的高层接口，使其容易使用。外观模式不会“封装”原有的接口，需要的人仍可以使用系统完整的功能。

### 例子

_Head First Design Patterns_ 第 7 章中的家庭影院的例子：用一个接口包装所有设备的操作。

### 参考

+ _Head First Design Patterns_ 第 7 章

## Proxy Pattern 代理模式

### 典型特征

对象A（代理对象）将功能委托给对象B（被代理对象），两个对象**接口相同**

### 例子

+ **远程代理**: Java RMI
+ **虚拟代理**: 对于一个创建开销大的对象（如需要从网上下载专辑封面的对象），当对象在创建前或创建中时，虚拟代理处理请求；当对象创建完成后，虚拟代理会直接将请求委托给真实对象
+ **保护代理**: 控制对原始对象的访问，只开放有访问权限的方法

`java.lang.reflect.Proxy` 可以创建动态代理，在运行时实现代理模式。

Spring AOP 基于动态代理实现。

### 参考

+ _Head First Design Patterns_

## Bridge Pattern 桥接模式

### 目的

将*抽象 (Abstraction)*（一个类）和*实现 (Implementor)*（一个类做的事情）解耦，使得两者可以独立地变化，而不会相互影响。

### 例子

桥接模式没有比较简单典型的例子，最好的例子还是 Wikipedia 中的 [Shape-drawing 的例子](https://en.wikipedia.org/wiki/Bridge_pattern#Java)。另外，_Head First Design Patterns_ 中给出了一个电视机-遥控器的例子。

+ Implementor: 接口 `DrawingAPI`，有方法 `drawCircle`
+ Concrete implementor: `DrawingAPI1`, `DrawingAPI2`，分别定义不同机制的绘图实现
+ Abstraction: 抽象类 `Shape`，保有 `DrawingAPI` 的引用，有方法 `draw`
+ Refined abstraction: 实现 `draw`，调用 `drawCircle`

有部分资料称 JDBC、SLF4J 使用了桥接模式，但有待商榷。讨论见 StackOverflow: ([1](https://stackoverflow.com/questions/46228420/why-is-jdbc-is-a-typical-example-of-bridge-design-pattern), [2](https://stackoverflow.com/questions/14888218/what-is-an-example-of-the-bridge-pattern-in-core-java))

有些情况下 implementor 退化为只有一个。

桥接模式可以看作是模板方法模式和策略模式的复合。

### 误区

桥接模式的应用范围比想象中的要窄。在很多资料中，桥接模式是**连接了两个维度**，避免两个维度的变化将导致组合爆炸。例子如如笔-颜色、手机品牌-手机功能、汽车品牌-手动挡自动挡。但这些说法没有佐证。


### 参考

+ _Head First Design Patterns_ 附录

## Flyweight Pattern 享元（蝇量）模式

### 目的

支持大量的细粒度的对象，节省内存

### 典型特征

创建对象时，返回一个缓存的实例

### 例子

+ JVM 的 integer cache
  + -128 ~ 127 （上限可设置）的整数对象会预先缓存，不会生成重复的对象
+ `String.intern()`
  + 返回 string pool 中的一份缓存

参考：[Design Patterns in the Real World: Flyweight](https://tamasgyorfi.net/2016/05/30/design-patterns-in-the-real-world-flyweight/)

## Composite Pattern 组合模式

TODO

## Strategy Pattern 策略模式

### 例子

_Head First Design Patterns_ 中鸭子游戏的例子：

困难：
如果使用继承来复用代码，会给一些子类添加上不必要的行为（如橡皮鸭不会飞）。
如果使用Flyable接口，会造成代码无法复用。

方法：
由于鸭子的行为（飞行行为，叫声行为）是多变的，为了分开变化和不变化的部分，我们将鸭子的两个行为取出来作为新的类。
这样，鸭子类不需要知道行为的实现细节，可以针对接口进行编程。

好处：
这些行为可以相互替换，也可以加入新的行为。

+ [**GOOD**] `java.util.Comparator`，传入`java.util.Arrays.sort()`（另有`java.util.List.sort()`与`java.util.Collections.sort()`，与`Arrays.sort()`实际相同）
+ `java.io.FileFilter`与`java.io.FilenameFilter`，传入`java.io.File.listFiles()`
+ `android.support.v7.widget.RecyclerView.LayoutManager`，有子类`LinearLayoutManager`, `GridLayoutManager`；用户可自定义子类，并通过`RecyclerView.setLayoutManager()`动态设置

## State Pattern 状态模式

API 中没有找到好的例子，但实际系统中应该经常能用到。

### 参考

+ _Head First Design Patterns_ 第 10 章

## Observer Pattern 观察者模式

Java 标准库中有 `java.util.Observable` 类和 `java.util.Observer` 类。但 `Observable` 类有明显的缺点，一般没有人用。可以使用 Guava 中的 `EventBus`。

### 参考

+ _Head First Design Patterns_ 第 2 章

## Command Pattern 命令模式

### 目的

将 (请求|方法调用) 封装在 `Command` 对象中。调用者只需要使用 `Command.execute()` 执行 (请求|方法调用)，而不需要知道执行的细节。

### 例子

两个非常典型的例子：

+ Java 多线程的 `Runnable` 和 `Callable` 都是 Command 类，线程池可以执行、调度不同的任务，而不需要关注任务的执行细节
  + 命令模式的好处：实现工作队列、任务调度
+ 数据库记录日志
  + 命令模式的好处：可撤销 (undo) 的命令，可重做 (redo) 的命令

### 参考

+ _Head First Design Patterns_ 第 6 章

## Mediator Pattern 中介模式

### 目的

用来解决对象间相互交互，相互耦合的问题。此模式定义了一个中介，封装了一系列对象间的交互行为。中介可以使得各对象不需要显示地相互交互。

TODO 

+ 中介模式 vs. 外观模式
+ 中介模式 vs. 观察者模式

### 例子

Java 线程池中的 `execute`, `submit`, `scheduleXxx` （Java 线程池是命令模式和中介模式的组合）

### 参考

+ [Mediator design pattern implemented in Java](https://github.com/mryndzionek/java_mediator)

## Iterator Pattern 迭代器模式

### 目的

可以顺序访问一个 collection 中的元素，而又不暴露内部的元素组织方式。

迭代器分为内部迭代器和外部迭代器。外部迭代器由 client 主动进行遍历，内部迭代器则是 client 提供回调函数，collection 内部进行遍历。Java 的 `Iterator` 是外部迭代器；`Stream.forEach` 可以看成内部迭代器。

### 例子

+ Java 中的 `Iterator`

### 参考

+ _Head First Design Patterns_ 第 9 章

## Interpreter Pattern 解释器模式

这个真的有应用场景吗？

### 例子

Java 正则表达式中的 `Pattern`

### 误区

解释器模式和 parser 无关，它不关注如何构造一个 AST。

## Template Method Pattern 模板方法模式

+ `java.io.InputStream`, `java.io.Reader` 中的 `read()` 等； `java.io.OutputStream`, `java.io.Writer` 中的 `write()` 等
+ `java.util.AbstractList` 中的 `add()`, `get()`, `remove()` 等；`java.util.AbstractMap` 中的 `put()`, `entrySet()`等
+ `android.app.Activity` 中的 `onCreate()`, `onResume()`, `onPause()` 等，提供默认行为，子类可扩充
+ `javax.servlet.http.HttpServlet` 中的 `doGet()`, `doPost()` 等，默认返回405错误，需要子类覆盖实现

### 参考

+ _Head First Design Patterns_ 第 8 章

## Chain of responsibility 责任链模式

### 目的

为了使多个 handler 都有机会处理请求，从而将请求的发送者和接受者解耦。将多个 handler 连成一条链，沿着这条链传递请求，每个 handler 可以选择是否将请求传递给下一个 handler。

### 例子

+ `okhttp3.Interceptor`，每一个 interceptor 都是一个 handler （[OkHttp 的 Interceptors 与责任链模式](http://nettee.github.io/posts/2018/OkHttp-Interceptors-and-Chain-of-Responsibility-Pattern/))
+ Logger 可以用责任链模式实现（但这不是一个推荐的实现方式）
+ Android 中对 TouchEvent 的处理（思路接近）

## Memento Pattern 备忘录模式

### 目的

在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，以便在适当的时候恢复对象 ("undo")。

+ _Originator_
+ _Caretaker_
+ _Memento_

### 例子

+ 伪随机数生成器
+ 有限状态自动机中的状态
+ `java.util.Date`，内部状态用 `long` 表示
+ Java 的可序列化对象

## Visitor Pattern 访问者模式

TODO

## 设计模式间的区分

+ Stategy vs. State: 见 _Head First Design Patterns_ 第10章
+ Chain of Responsibility vs. Decorator: [Why would I ever use a Chain of Responsibility over a Decorator?](https://stackoverflow.com/questions/747913/why-would-i-ever-use-a-chain-of-responsibility-over-a-decorator)
+ [那些相似的设计模式的区别](https://blog.csdn.net/jinzhuojun/article/details/11555595)

## 参考资料

+ _Head First Design Patterns_
+ GoF _Design Patterns_
+ [Wikipedia: Software design pattern](https://en.wikipedia.org/wiki/Software_design_pattern)
+ [图说设计模式](https://design-patterns.readthedocs.io/zh_CN/latest/index.html)

设计模式的对应例子：

+ Java 标准库里的设计模式
  +  [Examples Design Patterns in Java's core libraries](https://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns-in-javas-core-libraries).
  + [Patterns in Java APIs](http://cecs.wright.edu/~tkprasad/courses/ceg860/paper/node26.html)
+ Android 库里的设计模式
  + [Android 源码设计模式分析](https://github.com/simple-android-framework/android_design_patterns_analysis)
