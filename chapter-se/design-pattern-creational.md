# 设计模式 - 创建型模式

+ 创建型模式 (creational patterns)
  + Singleton 单例
  + Builder 构建（生成器）
  + Prototype 原型
  + Factory method 工厂方法
  + Abstract factory 抽象工厂

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