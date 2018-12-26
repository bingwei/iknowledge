# 面向对象 (Object-oriented Programming)

面向对象的三个基本特征：

+ 封装 (Encapsulation)
+ 继承 (Inheritance)
+ 多态 (Polymorphism)

## 类 (Class) vs 原型 (Prototype)

+ Class-based: C++, Java
+ Prototype-based: JavaScript, Io

## 多重继承 (multiple inheritance)

[Multiple inheritance - Wikipedia](https://en.wikipedia.org/wiki/Multiple_inheritance)

在《松本行弘的程序世界》书中，作者列举了多重继承的三个问题：

+ 结构复杂化
  + 如果是单一继承，一个类的父类是什么，父类的父类是什么，都很明确，因为只有单一的继承关系，然而如果是多重继承的话，一个类有多个父类，这些父类又有自己的父类，那么类之间的关系就很复杂了。
+ 优先顺序模糊
  + 假如我有 A，C 类同时继承了基类，B 类继承了 A 类，然后 D 类又同时继承了 B 和 C 类，所以 D 类继承父类的方法的顺序应该是 D、B、A、C 还是 D、B、C、A，或者是其他的顺序，很不明确。
+ 功能冲突
  + 因为多重继承有多个父类，所以当不同的父类中有相同的方法是就会产生冲突。如果 B 类和 C 类同时又有相同的方法时，D 继承的是哪个方法就不明确了，因为存在两种可能性。即**菱形问题 (Diamond problem)**

为了能够利用多继承的优点，同时解决多继承的问题，不同的语言有不同的方案：

+ 规格继承：只有声明，没有实现
  + Java 中的 _interface_ (7 以前)
+ 实现继承：_Mixin_
  + Ruby 中的 _Module_ ([Modules - Ruby Doc](https://ruby-doc.org/core-2.6/doc/syntax/modules_and_classes_rdoc.html))

Mixin 实际上是组合模式的一种体现 TODO ？

### C++ 的多继承

如果两个父类含有相同的 method，编译报错

解决方案：虚继承 (virtual inheritance)

### Java 的类单继承、接口多继承

由于 Java 的接口仅仅是声明 method，接口的多继承不会导致菱形问题。

然而，Java 8 引入 default method 之后，会出现菱形问题。

+ TODO Java 的 interface 在 JVM 中是什么样子的？

[Java 为什么不支持多继承？- 知乎](https://www.zhihu.com/question/24317891)
[C++ 多继承有什么坏处，Java 的接口为什么可以摈弃这些坏处？- 知乎](https://www.zhihu.com/question/31377101)

### Ruby 的 mixin