# 设计模式

如何学习设计模式：要能讲解出来，能举出例子。

## 设计模式的对应例子

Java 标准库里的设计模式： [Examples Design Patterns in Java's core libraries](https://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns-in-javas-core-libraries).

更多的例子：

### Strategy pattern

+ `java.io.FileFilter`与`java.io.FilenameFilter`，传入`java.io.File.listFiles()`

### Template method pattern

+ Android activity lifecycle (`onCreate()`, `onResume()`, `onPause()`, etc.)

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
