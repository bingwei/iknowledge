# Java 虚拟机 - 内存管理

## JVM 内存布局

![JVM memory](jvm/jvm-areas.png)

+ 线程私有
  + _Program counter_ 程序计数器
  + _Stack_, _VM stack_ 栈，虚拟机栈
    + _Frame_ 栈帧
      + _Local variable_ 局部变量
      + _Operand stack_ 操作数栈
      + 返回值
      + 指向当前方法的类的**运行时常量池**的引用
  + _Native method stack_ 本地方法栈
+ 线程共享
  + _Heap_ 堆
  + _Permanent generation_ 永久代
    + Interned strings
    + _Method area_ 方法区

JVM 是**基于栈**的，使用操作数栈 (operand stack)，而不是**基于寄存器**的。注意：**每个栈帧都包含一个操作数栈！**

### Interned string

TODO

### 方法区 Method Area

等价于 Linux 进程中的 **text 段**（只读代码、只读数据）。

方法区存储 _per-class structures_，包括：

+ 运行时常量池
+ Field data
+ Method data
+ 方法（包括 `<init>` 与 `<clinit>`）的代码

### 运行时常量池 Run-time constant pool

类似于 C 语言中的**符号表**，但内容更加丰富。

每个类有一个运行时常量池，对应 class 文件中的 `constant_pool`。

### 动态链接 Dynamic linking

> Dynamic linking translates these symbolic method references into concrete method references, loading classes as necessary to resolve as-yet-undefined symbols, and translates variable accesses into appropriate offsets in storage structures associated with the run-time location of these variables.

动态连接是方法的**符号引用**解析为方法的**具体引用**的过程。同一方法调用的动态链接只会进行一次。

### StackOverflowError (SO) 与 OutOfMemoryError (OOM)

+ `StackOverflowError` —— 栈区已满，无法放入新的栈帧
  + 一般都是由不正确的无限递归导致的
  + 既可能是 VM stack 也可能是 Native method stack
+ `OutOfMemoryError` —— 堆区已满，无法为新的对象分配空间
  + 不仅是堆区，其他区域满了也可能 OOM
  + 如果栈是动态扩展的，栈也可能会 OOM（不过栈一般还是 SO）

除了 PC 之外，其他的内存区域都有可能出现 OOM。例如：

+ 堆溢出 `java.lang.OutOfMemoryError: Java heap space`
+ 永久代溢出 `java.lang.OutOfMemoryError: PermGen space`
  + 可能是方法区溢出或是字符串常量池溢出

## 垃圾回收 Garbage collection

![Java GC arguments](jvm/java-gc-args.png)

### 堆内存布局

+ _Young generation_ 新生代
  + _Eden_
  + _Survivor_： 分为 S0 和 S1
+ _Old generation_, _tenured generation_ 老年代
+ _Permanent generation (Perm Gen)_ 永久代
  + 注意 PermGen 并不属于堆区，划分在 _non-heap_ 中。

Static fields 和 static methods 是在永久代中

Java 8 开始废除永久代，改为 _Metaspace_（元空间）。永久代使用堆内存空间，而 metaspace 使用的是物理内存（native memory）。

内存参数：

+ `-Xms` —— 堆的大小下限、初始大小
+ `-Xmx` —— 堆的大小上限
  + 控制的是 YoungGen + OldGen，注意**不包括 PermGen**
+ `-Xmn` —— 年轻代的大小
+ `-XX:PermSize` —— （Java 7 以前）永久代的大小下限、初始大小
+ `-XX:MaxPermSize` —— （Java 7 以前）永久代的大小上限
+ `-XX:MetaspaceSize` —— （Java 8 以后）元空间的大小

### 判断已死亡对象

+ 引用计数法
  + 实现简单
  + 判定效率高
  + 很难解决循环引用的问题
+ 可达性分析法
  + 以 _GC roots_ 作为起始点
  + 不可达的对象则已死亡

### 垃圾回收策略

+ 新的对象创建在新生代
+ _Minor garbage collection_ 会在新生代进行
+ _Major (full) garbage collection_ 会将仍存活的新生代对象移动至老年代
  + 一般会使应用程序的线程暂停
+ 老年代和永久代其中任何一个满了，两个都会进行垃圾回收

### 垃圾回收算法

根据三代的不同特点，使用不同的垃圾回收算法。

+ 新生代 —— 对象存活率较低 —— 使用**复制算法**
+ 老年代 —— 对象存活率较高 —— 使用**标记-清理算法**、**标记-整理算法**

#### Copying 复制算法

+ 内存划分：
  + 普通版：划分为大小相等的两块
  + 实际版：按 8:1:1 划分为 Eden, Survivor 0 和 Survivor 1
+ 每次使用 eden 和其中一个 survivor。垃圾回收时，将存活对象复制到另一个 survivor 中
+ 如果存活对象多于 10%，需要进行 _handle promotion_（分配担保）

#### Mark-sweep 标记-清理算法

+ 工作方式
  + 标记阶段：标记出需要回收的对象
  + 清理阶段：回收被标记的对象
+ 标记和清理的效率都不太高
+ 会产生大量不连续的内存碎片
  + 大对象可能无法分配空间，还需要额外 GC

#### Mark-compact 标记-整理算法

+ 工作方式
  + 标记阶段：与“标记-清理算法”相同
  + 整理阶段：将所有存活的对象移动到一边，然后直接清理到边界以外的内存

### 垃圾回收器

TODO

![JVM garbage collector pairs](jvm/jvm-garbage-collector-pair.png)

一文学习：[Java虚拟机垃圾回收(三) 7种垃圾收集器：主要特点 应用场景 设置参数 基本运行原理](https://blog.csdn.net/tjiyu/article/details/53983650)

### 关于永久代

永久代包括：

+ 方法区
+ Interned strings

永久代是 _non-heap memory_ 的一部分。

动态生成类的情况，容易造成永久代的 OutOfMemoryError，例如反射、动态代理、动态生成 JSP。

## Java 对象内存布局

### 对象内存布局

+ _Object header_ 对象头
  + _Mark word_ (4B on 32-bit-arch, 8B on 64-bit-arch)
    + 保存了锁的状态（因此每个 Java 对象都可以作为锁）
  + _Klass pointer_ (4B on 32-bit-arch, 4B/8B on 64-bit-arch)
    + 指向对象的类的元数据的指针
+ Instance data 实例数据
  + 各种 field
+ Padding 对齐填充
  + 按 8 字节对齐

![Mark word in Java object header](jvm/object-header-mark-word.png)

### 对象的创建过程

对象的创建过程在字节码层面，需要先使用 `new` 指令创建内存空间，再调用 `<init>` 方法。`new` 执行分为四步创建对象：

+ 类加载检查
  + 如果未加载，则加载
+ 分配内存
+ 初始化零值
  + 保证 field 不需要赋初值就能直接使用
  + 对象头不赋值
+ 设置对象头