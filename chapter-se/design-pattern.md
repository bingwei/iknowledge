# 设计模式

如何学习设计模式：要能讲解出来，能举出例子。

参考文档：
+ [Wikipedia: Software design pattern](https://en.wikipedia.org/wiki/Software_design_pattern)
+ [图说设计模式](https://design-patterns.readthedocs.io/zh_CN/latest/index.html)

## 设计模式的对应例子

+ Java 标准库里的设计模式： [Examples Design Patterns in Java's core libraries](https://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns-in-javas-core-libraries).
+ Android 库里的设计模式： [Android 源码设计模式分析](https://github.com/simple-android-framework/android_design_patterns_analysis)

更多的例子：

### Strategy pattern

+ `java.io.FileFilter`与`java.io.FilenameFilter`，传入`java.io.File.listFiles()`
+ `java.util.Comparator`，传入`java.util.Arrays.sort()`（另有`java.util.List.sort()`与`java.util.Collections.sort()`，与`Arrays.sort()`实际相同）
+ `android.support.v7.widget.RecyclerView.LayoutManager`，有子类`LinearLayoutManager`, `GridLayoutManager`；用户可自定义子类，并通过`RecyclerView.setLayoutManager()`动态设置

### Command pattern

+ `java.lang.Runnable`与`java.util.concurrent.Callable`，交给`ExecutorService`执行
+ 数据库记录日志，用来undo, redo

### Template method pattern

+ `java.io.InputStream`, `java.io.Reader` 中的 `read()` 等； `java.io.OutputStream`, `java.io.Writer` 中的 `write()` 等
+ `java.util.AbstractList` 中的 `add()`, `get()`, `remove()` 等；`java.util.AbstractMap` 中的 `put()`, `entrySet()`等
+ `android.app.Activity` 中的 `onCreate()`, `onResume()`, `onPause()` 等，提供默认行为，子类可扩充
+ `javax.servlet.http.HttpServlet` 中的 `doGet()`, `doPost()` 等，默认返回405错误，需要子类覆盖实现

### Chain of responsibility pattern

+ 很多logger，如`java.util.logging.Logger`（有待验证）
+ `okhttp3.Interceptor`
+ Android中对TouchEvent的处理（思路接近）

## 设计模式间的区分

+ [那些相似的设计模式的区别](https://blog.csdn.net/jinzhuojun/article/details/11555595)
+ Chain of Responsibility vs. Decorator: [Why would I ever use a Chain of Responsibility over a Decorator?](https://stackoverflow.com/questions/747913/why-would-i-ever-use-a-chain-of-responsibility-over-a-decorator)

## 设计模式学习笔记

### 策略模式 Strategy pattern

_Head First Design Patterns_ 中鸭子游戏的例子：

困难：
如果使用继承来复用代码，会给一些子类添加上不必要的行为（如橡皮鸭不会飞）。
如果使用Flyable接口，会造成代码无法复用。

方法：
由于鸭子的行为（飞行行为，叫声行为）是多变的，为了分开变化和不变化的部分，我们将鸭子的两个行为取出来作为新的类。
这样，鸭子类不需要知道行为的实现细节，可以针对接口进行编程。

好处：
这些行为可以相互替换，也可以加入新的行为。
