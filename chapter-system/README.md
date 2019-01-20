# 计算机系统

## 数据表示

+ 整数的表示，补码
+ 浮点数的表示，IEEE 754
+ ASCII 与 UTF-8 编码
+ 大端 & 小端
  + [大小端存储模式精解 - Jocent Zhou](https://jocent.me/2017/07/25/big-little-endian.html)
+ 位运算
+ 整数的加减与溢出标识 CF, OF, SF, ZF

## 汇编语言

+ 汇编的寻址方式，`mov` 指令和 `lea` 指令的联系与区别
+ 过程调用中的栈帧变化，入口参数，局部变量，`call` / `ret`
+ If, switch, while 结构的汇编表示
+ 汇编语言如何跳转，`cmp` / `test` 与 jump 指令

## 链接

### 目标文件

目标文件 (object file) 有三种形式（都是 ELF 文件）：

+ 可重定位目标文件 (relocatable object file)
  + 由单个模块（源文件）生成
  + 代码区/数据区总是从地址 0 开始
+ 可执行目标文件 (executable object file)
  + 代码区/数据区的地址为 0x8048xxx
+ 共享目标文件 (shared object file)，即动态链接库

符号表 (.symtab section)，包含模块中定义或引用的符号，有三种符号：

+ 全局符号，可被其他模块引用
+ 外部符号，引用其他模块
+ 本地符号，只能模块内引用
  + `static` 修饰的函数（模块私有函数）
  + `static` 修饰的全局变量（模块私有全局变量）
  + `static` 修改的局部变量（静态局部变量）
  + **C 语言中 `static` 关键字的公共特性：都在 ELF 符号表中出现，且都是本地符号。**

### 链接 (Link)

链接的分类

+ 目标文件
+ 目标文件 + 静态库 (.a 即 archive)
+ 目标文件 + 共享库 (.so 即 shared object)
  + 静态链接阶段：重定位和符号表信息
  + 动态链接阶段：代码和数据

静态链接的两个主要任务：

+ 符号解析 (symbol resolution)：找到符号引用对应的符号定义
  + 对于多重定义的全局符号，使用强弱性区分
  + 与静态库链接时，根据命令行参数顺序进行解析
+ 重定位 (relocation)：定位符号的实际地址
  1. 将 section 合并，确定指令、全局变量的实际地址
  1. 修改符号的引用，确定符号引用的实际地址

动态链接，生成位置无关代码 (PIC)。本质上是在数据段使用 GOT 表，在运行时确定目标代码的位置。有点“将代码看成数据”的感觉。

### 加载 (Load)

加载器的三个步骤

+ `execve` 调用加载器
+ 拷贝代码段 (.init, .text, .rodata) 和数据段 (.data, .bss)
+ 跳转到 _entry point_ (`_start`)
  + 依次调用：
    + `__libc_init_first`
    + `_init`
    + `atexit`
    + `main`
    + `_exit`

## 缓存

### 命中与不命中

+ Cache hit 命中
+ Cache miss 不命中
  + Cold miss 冷不命中 —— 缓存是空的
  + Conflict miss 冲突不命中 —— 两个数据来回替换
    + 即 _抖动（thrash)_
  + Capacity miss 容量不命中 —— 缓存满了

### 缓存步骤

+ 组选择 (set selection)
+ 行匹配 (line matching)
  + 检查有效位，比较 tag，判断是否命中
+ 字抽取，字选择 (word extraction, word selection)
  + 得到 block 之后，根据 block offset 获取一些字节

### 缓存类型

+ 直接映射 (direct-mapped)：每组只有一行
+ 组相联 (set associative)：每组有多行
+ 全相联 (fully associative)：只有一个组

| 缓存类型 | 组选择 | 行匹配 | 行替换
| :-: | :-: | :-: | :-: | :-: |
| 直接映射 | 根据组索引 (set index) | 组中只有一行，直接匹配 | 直接替换 |
| 组相联 | 根据组索引 | 搜索组中每一行 | LFU，LRU 等 |
| 全相联 | 不需要选择组（只有一个） | 搜索组中每一行 | LFU，LRU 等 |

注意：

+ 直接映射缓存更容易引起 conflick miss，即抖动问题
+ 全相联缓存由于搜索开销大（命中时间长），只适合小缓存
  + 例如，快表 TLB
+ 相联度越高，命中率越高，命中时间越长
  + 权衡问题 trade-off

## 虚拟存储器

虚拟存储器的好处：

+ 存储管理
  + 简化链接器的设计
    + .text 总是从 0x8048000 开始
  + 简化加载器的设计
    + 总是加载到 0x8048000
+ 存储保护
  + 页表项上的标志位可以控制读写权限

## 中断与异常

系统调用都是 `int 0x80` 异常，调用号放在 `%eax`，参数放在 `%ebx`, `%ecx`, `%edx`, `%esi`, `%edi`, `%ebp`，最多六个参数。

异常使得操作系统可以实现上下文切换 (context switch)，从而实现进程的抢占式调度。