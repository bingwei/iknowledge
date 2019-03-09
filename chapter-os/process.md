# 操作系统 - 进程

## 进程

### 虚拟地址空间

+ 代码段 text segment
+ 数据段（分为初始化的和未初始化的）
+ 堆
+ 栈
+ 内核区

非常重要，图参考 _The Linux Programming Interface_ Figure 6-1

### 进程复制

+ fork
+ vfork
+ clone

fork 是将进程复制了一份。_The child process is an exact duplicate of the parent process_.（有一些例外，比如 PID）。但是一般会进行一些优化：

+ 代码段 text segment
  + 由于是只读的，实际上没有复制，只是复制了页表项，指向相同的内存区域
+ 数据段、堆、栈
  + copy on write，当父子任意一进程要修改内容的时候，复制要修改的页

所以，fork 之后父子进程相同的内容：

+ 代码段 —— 只读数据
+ 已打开文件的描述符 —— 复制了 _file descriptor table_，但是指向 open file table 中相同的项
  + 参考 _The Linux Programming Interface_ Figure 24-2

Linux 3.2 提供了 clone 系统调用，可以控制哪些部分由父子进程共享。库函数 `fork()` 和 `clone()` 都调用了系统调用 `clone`。参考 [clone 的 man page](http://man7.org/linux/man-pages/man2/clone.2.html)，其中的 `flags` 部分是可以设置为**共享**的，而 `fork()` 这些都不共享，而是**复制**。

（[Is it true that fork() calls clone() internally?](https://stackoverflow.com/questions/18904292/is-it-true-that-fork-calls-clone-internally)）

### 孤儿进程与僵尸进程

参考 _The Linux Programming Interface_ Section 26.2

+ [孤儿进程与僵尸进程[总结]](https://www.cnblogs.com/Anker/p/3271773.html)

## 进程间通信 IPC

+ 通过打开的文件
  + fork 之后父子进程共享打开的文件
+ 管道
  + 匿名管道 PIPE
  + 命名管道 FIFO
+ 网络通信接口：Socket
+ 共享内存 shared memory
  + POSIX 共享内存 —— `shm_open` / `mmap`
  + System V 共享内存 —— `shmget` / `shmat`
  + `mmap` / `mmap`
+ 信号量 semaphore
+ 消息队列 message queue（现已几乎不用）
+ 信号 signal

使用场景：

+ Socket 是第一选择，适合分布式系统
+ 两个进程间通信，适合用管道（包括 PIPE 和 FIFO）
+ 多个进程间通信，适合用共享内存配合信号量

### 管道 PIPE

用到了 `pipe()` 和 `dup2()`。

也可以使用包装过的函数 `popen()` 和 `pclose()`。（还记得 Python 中的 `Popen` 吗？）

参考：[一个小小的 Shell 管道符，内部实现可真不简单！](https://juejin.im/post/5bc98b36f265da0af93b34c6)

### Socket 通信（网络 IPC）

Socket 是最通用的进程间通信方式，而且是在不同机器间通信的首选方式。参考[网络编程](../chapter-network/socket.md)章节。
