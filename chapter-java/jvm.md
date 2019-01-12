# Java 虚拟机

TODO list:

+ Java 与 JVM 的关系，基于 JVM 的语言
+ JVM 内容 [JVM Internals](http://blog.jamesdbloom.com/JVMInternals.html)
  + Native method
  + JVM 的执行过程
  + JIT
+ 垃圾回收
  + JVM 参数
+ 类加载器
  + 双亲委派模型

JVM 是**基于栈**的，使用操作数栈 (operand stack)，而不是**基于寄存器**的。

## 重要组件

+ PC
+ 栈 (stack)
  + 栈帧 (frame)
    + 局部变量 (local variable)
    + 操作数栈 (operand stack)
      + 注意：**每个栈帧都包含一个操作数栈！**
+ 堆 (heap)
+ Method area

## 字节码 Bytecode

基本指令：

+ `iconst` 将整数常量放在 operand stack 上
+ `istore` 从 operand stack 上 pop 一个值存储在 local variable 中
+ `iload` 从 local variable 中加载一个值，push 到 operand stack 上

函数调用指令：

+ `invokestatic` 调用静态方法
+ `invokevirtual` 调用普通方法
  + 需要传递对象实例作为第一个参数（成为方法中的 `this`）
+ `invokespecial` 调用特殊方法
  + `<init>`
+ `return` 返回

参数传递：

+ 调用者将参数按顺序（从左至右依次 push）放在 operand stack 上
+ 被调用者从 local variable array 中获取参数
+ 被调用者将返回值放在 operand stack 上
+ 调用者从 operand stack 上获取返回值

对象创建：

+ 使用 `new` 在堆区创建内存空间，返回对象引用
+ 使用 `dup` 复制对象引用
+ 使用 `invokespecial <init>` 调用 constructor
  + 第一个参数需要是对象引用
  + 无返回值

IA-32 汇编指令与 Java 字节码指令的对比

| | 汇编 | 字节码 |
| :-: | :-: | :-: |
| 模型 | 基于寄存器 | 基于操作数栈 (operand stack)（大小在编译时确定）|
| 局部变量保存位置 | 寄存器，寄存器放不下放在栈中 | 局部变量数组（大小在编译时确定）|
| 指令长度 | 变长 | 变长 |
| 函数调用指令 | `call` / `ret` | `invoke*` / `return` |
| 栈帧管理 | 函数一开始保存 `%ebp` 旧值，以及移动 `%esp` | 自动管理，包括传递操作数栈或局部变量数组的内容 |
| 参数传递 | 调用者放在自己的栈帧底部，被调用者从调用者栈帧获取 | 调用者放在自己的操作数栈，会自动拷贝到被调用者的局部变量数组，被调用者从自己的局部变量数组中获取 |
| 返回值传递 | 一般放在 `%eax` 寄存器中 | 被调用者放在自己的操作数栈，会自动拷贝到调用者的操作数栈，调用者从自己的操作数栈中获取 |

## 垃圾回收 Garbage collection

TODO

![Java GC arguments](jvm/java-gc-args.png)

## 参考

+ _The Java Virtual Machine Specification, Java SE 7 Edition_
+ _The Java Virtual Machine Specification, Java SE 8 Edition_
+ 周志明，《深入理解 Java 虚拟机，第 2 版》
+ [JVM Internals](http://blog.jamesdbloom.com/JVMInternals.html)