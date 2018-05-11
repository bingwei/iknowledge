# C++ 中的类型转换 (cast)

在 C 语言中，类型转换只有一种写法 `(type) expression`，但是可能有多种语义：

+ 在两种算数类型间转换
+ 在整数类型/指针类型间转换
+ 在两种指针类型间转换
+ 为类型添加/去除 cv-qualifiers

C++ 中为了区分类型转换的不同语义，引入了四种新的类型转换的写法，分别有不同的作用：

+ `const_cast<type>(expression)` 用来添加/去除变量的 `const` qualifier
+ `dynamic_cast<type>(expression)` 用在有继承（多态）的情况下，实现 **safe downcasting**
+ `static_cast<type>(expression)` 和 C 中传统的类型转换最接近，用来实现很多的隐式类型转换
+ `reinterpret_cast<type>(expression)` 比较复杂，同时也不常用

参考资料：

+ 一个容易理解的回答：[How do you explain the differences among static_cast, reinterpret_cast, const_cast, and dynamic_cast to a new C++ programmer?](https://www.quora.com/How-do-you-explain-the-differences-among-static_cast-reinterpret_cast-const_cast-and-dynamic_cast-to-a-new-C++-programmer)
+ 更多内容请参考 _More Effective C++_ : "Item 2"
