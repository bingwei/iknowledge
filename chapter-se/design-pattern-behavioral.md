# 设计模式 - 行为型模式

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
