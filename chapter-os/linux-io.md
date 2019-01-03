# Linux I/O

## Socket 编程

send / recv 见 APUE 第 16 章。

## Linux 的 IO 模型

五种典型的 I/O 模型：

+ 阻塞 I/O (Blocking I/O, BIO)
  + 最常用，最简单，默认模型
+ 非阻塞 I/O (Non-blocking I/O, NIO)
+ 多路复用 I/O (Multiplexing I/O)
  + 也叫 select/poll
+ 信号驱动式 I/O (Signal-driven I/O, SIGIO)
  + 不太重要
+ 异步 I/O (Asynchronous I/O, AIO)
  + `aio_read` 或 libevent 函数库
  + 前三个都可以叫做同步 I/O

一次 IO 包括两个阶段：

1. 等待数据（到达内核缓冲区）：此时是 IO 密集阶段
1. 将数据从内核缓冲区复制到进程缓冲区：此时是 CPU 密集阶段

| I/O 模型 | 第一阶段 | 第二阶段 |
| :-: | :-: | :-: |
| BIO | 阻塞 | 阻塞 |
| NIO | 非阻塞（需要轮询）| 阻塞 |
| Select/poll | 阻塞 | 阻塞 |
| AIO | 非阻塞 | 非阻塞 |

其中 Select/poll 虽然和 BIO 都是阻塞，但 select/poll 可以同时处理多个 I/O。

![Five I/O models](io-models.png)

参考：

+ [聊聊同步、异步、阻塞与非阻塞](https://www.jianshu.com/p/aed6067eeac9)
+ [聊聊 Linux 五种 IO 模型](https://www.jianshu.com/p/486b0965c296)