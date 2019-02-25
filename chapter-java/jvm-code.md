# Java 虚拟机 - 代码执行

## Class 文件格式

TODO

常量池：主要存放两大类常量：字面量和符号引用。

## 字节码指令 Bytecode instructions

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

上锁：

+ Case 1: synchronized method
  + 方法添加 `ACC_SYNCHRONIZED` 标识
+ Case 2: `synchronized (obj)`
  + 在同步块的开始/结束位置使用 `monitorenter` / `monitorexit` 指令
  + 要上锁的对象放在栈顶

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

## 类加载

https://en.wikipedia.org/wiki/Java_Classloader
  
TODO 双亲委派模型