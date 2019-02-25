# Java 虚拟机 - 内存管理

## JVM 内存布局

![JVM memory](jvm/jvm-memory.png)

+ 线程私有
  + _Program counter_ 程序计数器
  + _Stack_, _VM stack_ 栈，虚拟机栈
    + _Frame_ 栈帧
      + _Local variable_ 局部变量
      + _Operand stack_ 操作数栈
      + ...
  + _Native method stack_ 本地方法栈
+ 线程共享
  + _Heap_ 堆
  + _Method area_ 方法区

JVM 是**基于栈**的，使用操作数栈 (operand stack)，而不是**基于寄存器**的。注意：**每个栈帧都包含一个操作数栈！**

### StackOverflowError 与 OutOfMemoryError

+ `StackOverflowError` —— 栈区已满，无法放入新的栈帧
  + 既可能是 VM stack 也可能是 Native method stack
+ `OutOfMemoryError` —— 堆区已满，无法为新的对象分配空间

## 垃圾回收 Garbage collection

![Java GC arguments](jvm/java-gc-args.png)

### 堆内存布局

+ _Young generation_ 新生代
  + _Eden_
  + _S0_, _Survivor 0_
  + _S1_, _Survivor 1_
+ _Old generation_ 老年代
+ _Perm_ 永久代

Java 8 开始废除永久代，改为 _Metaspace_（元空间）。永久代使用堆内存空间，而 metaspace 使用的是物理内存。

内存参数：

+ `-Xms` —— 堆的初始大小
+ `-Xmx` —— 堆的最大大小
+ `-Xmn` —— 堆中年轻代的大小
+ `-XX:PermSize` —— 永久代的初始大小，最小大小
+ `-XX:MaxPermSize` —— 永久代的最大大小

### 判断已死亡对象

+ 引用计数法
  + 实现简单
  + 判定效率高
  + 很难解决循环引用的问题
+ 可达性分析法
  + 以 _GC roots_ 作为起始点
  + 不可达的对象则已死亡

### 垃圾回收策略

分配策略：

+ 对象优先在 eden 区分配
+ 大对象直接进入老年代
+ 长期存活的对象将进入老年代

GC 分为 minor GC 和 major GC：

+ _Minor GC_
  + 发生在新生代的 GC
+ _Major GC_, _Full GC_
  + 发生在老年代的 GC

### 垃圾回收算法

JVM 将堆区分为三代，根据三代的不同特点，使用不同的垃圾回收算法。

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

分为五步：

+ 类加载检查
  + 如果未加载，则加载
+ 分配内存
+ 初始化零值
  + 保证 field 不需要赋初值就能直接使用
  + 对象头不赋值
+ 设置对象头
+ 执行 `<init>` 方法