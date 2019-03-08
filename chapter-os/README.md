# 操作系统

## 文件

### inode

inode 全名是 _index node_。

inode 中保存文件的各种信息，唯独**不包括文件名**。Linux 使用 inode 号码来识别文件，而不是文件名。

如何从文件名获取 inode 号码？目录文件中存储了文件名和 inode 号码的对应关系。由于目录文件内只有文件名和inode号码，所以如果只有读权限，只能获取文件名，无法获取其他信息，因为其他信息都储存在inode节点中，而读取inode节点内的信息需要目录文件的执行权限（x）。

如何从 inode 号码获取文件名？

参考：

+ _The Linux Programming Interface_, Section 14.4
+ [理解inode - 阮一峰](http://www.ruanyifeng.com/blog/2011/12/inode.html)

### 文件描述符 file descriptor

文件描述符是非负整数，描述**打开的文件**。

+ 0 —— stdin 标准输入
+ 1 —— stdout 标准输出
+ 2 —— stderr 标准错误

Unix 内核会为每个进程维护一个**文件描述符表** (_file descriptor table_)。文件描述符则是这个表的 index，指向**打开文件表** (_open file table_) 中的一项。通过 `dup` / `dup2` 函数，两个文件描述符可以指向同一个打开文件表项。

打开文件表记录了文件的 offset 和 flags（`open()` 的参数），以及指向 _inode table_ 中的一项。一个文件可以被打开多次，有多个打开文件表项，它们的 offset 和 flags 不同，但指向同一个 inode。

![File descriptor table, open file table and inode table](img/file-descriptor-table.png)

（图片来自 _The Linux Programming Interface_）详情参考书的 5.4 节。

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

## Socket 通信（网络 IPC）

服务器先用 `listen()` 表明愿意接收请求，接下来：

+ 用 `accept()` 阻塞等待
+ 用 `select()` / `poll()` / `epoll()` 多路等待

## 缓存替换策略

LRU 如何实现，以及 Redis 中如何实现 LRU：https://zhuanlan.zhihu.com/p/34133067

Apache commons 中有一个 LRU map