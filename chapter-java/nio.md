# NIO

Java 的 NIO 实际上对应 I/O 模型中的**多路复用 I/O**。

## BIO vs. NIO

BIO 是 _面向流_ 的，一次一个字节地处理数据。模型简单，但是速度较慢。

NIO 是 _面向缓冲区_ 的，每一个操作都在一步中产生或者消费一个数据块。按块处理数据比按 (流式的) 字节处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

BIO 模型由于是阻塞的，必须依赖线程进行加速。然而，当连接数量众多的时候，线程的创建、维护、切换成本很高。此时 NIO 比 BIO 更加高效。

NIO 一个重要的特点是：socket 主要的读、写、注册和接收函数，在等待就绪阶段都是非阻塞的，真正的 I/O 操作是同步阻塞的（耗时较短）。NIO 由原来的阻塞读写（占用线程）变成了单线程轮询事件，找到可以进行读写的网络描述符进行读写。除了事件的轮询是阻塞的（没有可干的事情必须要阻塞），剩余的 I/O 操作都是纯 CPU 操作，没有必要开启多线程。

## Java NIO

TODO

> JDK1.4中新加入的 NIO(New Input/Output) 类，引入了一种基于通道（Channel） 与缓存区（Buffer） 的 I/O 方式，它可以直接使用Native函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆之间来回复制数据。

如何理解这段话？

### 概念

_通道 (Channel)_ 和 _缓冲区 (Buffer)_ 是 NIO 中的核心对象，几乎在每一个 I/O 操作中都要使用它们。

Channel 与 stream 类似，不过 channel 是双向的。NIO 中，通过 channel 来读/写数据。Buffer 是一个容器（块）。读数据时，数据从 channel 读入 buffer；写数据时，数据从 buffer 写入 channel。不能**直接**将字节写入 channel，或者从 channel 中读出字节。（在 BIO 中，数据是直接写入 stream，或者直接从 stream 中读出。）

### Buffer

最常用的 buffer 是 `ByteBuffer`。

Buffer 的模型：position, limit, 和 capacity。

### Channel

四种 channel:

+ `FileChannel` 文件 I/O
+ `DatagramChannel` UDP
+ `SocketChannel` TCP (client)
+ `ServerSocketChannel` TCP (server)

其中 `FileChannel` 只能工作在阻塞模式下，而其他 channel 可以在阻塞/非阻塞模式中切换。

### Selector

使用 selector 可以在单线程中处理多个 channel。只有非阻塞模式下的 channel 才能够和 selector 一起使用。（也就是说 `FileChannel` 不能和 selector 一起使用。）

### 内存映射文件

TODO：内存映射文件

+ [Java NIO分析(11): 零拷贝技术以及NIO的支持](http://sound2gd.wang/2018/07/24/Java-NIO%E5%88%86%E6%9E%90-11-%E9%9B%B6%E6%8B%B7%E8%B4%9D%E6%8A%80%E6%9C%AF/)

## Netty

参考官方文档学习：

+ [User Guide (4.x)](https://netty.io/wiki/user-guide-for-4.x.html)