# Java IO

IO 分为阻塞型 IO (Blocking I/O, BIO)，非阻塞型 IO (Non-blocking I/O, NIO)，和异步 IO (Asynchronous I/O, AIO)。Java 标准库一开始只支持 BIO，在 1.4 版本引入 NIO，因此 NIO 又称为 "New IO"。Java 7 引入 NIO.2，支持 AIO。

## BIO & 流 (Stream)

| | 字节流 | 字符流 |
| :-: | :-: | :-: |
| 输入流 | `InputStream` | `Reader` |
| 输出流 | `OutputStream` | `Writer` |

+ 根据流的方向，流可分为 *输入流* 和 *输出流*
+ 跟据流中的内容，流可分为 *字节流* 和 *字符流*
+ 根据 IO 流中数据的来源，流可分为 *节点流（原始流）* 和 *链接流（装饰流）*

### 字节流

InputStream

+ `read()`
+ `read(byte[])`
+ `read(byte[], int, int)`
+ `skip(long)`
+ `available()`
+ `close()`
+ `mark(int)`, `reset()`, `markSupported()`

OutputStream

+ `write()`
+ `write(byte[])`
+ `write(byte[], int, int)`
+ `flush()`
+ `close()`

基本的节点流：

+ `FileInputStream` / `FileOutputStream` 从文件读/写数据
  + 大部分是 native 方法
+ `ByteArrayInputStream` / `ByteArrayOutputStream` 从字节数组读/写数据
+ <del>`StringBufferInputStream` 从字符串读数据</del> (deprecated)
  + 现在应该使用 `StringReader`

基本的链接流（`FilterInputStream` / `FilterOutputStream` 的子类）：

+ `BufferedInputStream` / `BufferedOutputStream` 提供缓存功能
+ `DataInputStream` / `DataOutputStream` 将字节数据转换为 Java 原始类型变量
+ `ObjectInputStream` / `ObjectOutputStream` 将字节数据转换为 Java 对象
  + 虽然不是 `FilterInputStream` 的直接子类，但内部用到了它的子类

以输入流为例（输出流同理），链接流中都会包含另一个 input stream，使用它的数据源（即将 `read` 委托给另一个 input stream），并进行一些数据转换的工作（`DataInputStream`, `ObjectInputStream`）或提供额外的功能（如`BufferedInputStream`）。这是设计模式-**装饰者模式**。

使用方式：

```Java
String filename = "a.txt";
InputStream is = new InputStream(filename);
BufferedInputStream bis = new BufferedInputStream(is);
```

![InputStream-OutputStream correspondence](io/InputStream-OutputStream-correspondence.jpg)

### 字符流

基本的节点流：

+ `FileReader` / `FileWriter` 从文件读/写数据 （利用 `InputStreamReader` / `OutputStreamWriter`）
+ `CharArrayReader` / `CharArrayWriter` 从字符数组读/写数据
+ `StringReader` / `StringWriter` 从字符串读/写数据

基本的链接流（`FilterReader`, `FilterWriter` 的子类）：

+ `BufferedReader` / `BufferedWriter` 提供缓存功能

![Reader-Writer correspondence](io/Reader-Writer-correspondence.jpg)

### 字节流与字符流的转换

+ `InputStreamReader`
+ `OutputStreamWriter`

这是设计模式-**适配器模式**。

### 类库细节

Standard I/O:

+ stdin -- `System.in`
+ stdout -- `System.out`
+ stderr -- `System.err`

`System.out` 和 `System.err` 都是`PrintStream` 对象，支持 print 系列函数。而 `System.in` 却是原始的 `InputStream`。因此在使用 `System.in` 之前需要进行必要的包装（例如用 `BufferedInputStream` 添加缓存，或用 `InputStreamReader` 适配为字符流）。

`RandomAccessFile` 是一个特立独行的类。

`Scanner` 和 `PrintWriter` 是常用了两个输入/输出工具类。`PrintWriter` 是 `Writer` 的子类，添加了对各种 print 函数的支持。`Scanner` 在 `java.util` 中，内部使用了 IO 类库。

## NIO

见 [NIO 章节](nio.md)。

## AIO

TODO

## Java I/O 编程

参考[官方 tutorial](https://docs.oracle.com/javase/tutorial/essential/io/file.html)，以 `java.nio.file.Path` 为核心，灵活使用 stream 与 channel。

## Java IO 类库发展历史

+ Java 1.0
  + `InputStream`, `OutputStream`
+ Java 1.1
  + `Reader`, `Writer`
+ Java 1.4
  + NIO
+ Java 7
  + NIO.2 (AIO)
  + `Path`

TODO: NIO.2

## 参考资料

+ [Java NIO 系列教程 (译)](http://ifeve.com/overview/)
+ [Java NIO 浅析 - 美团技术团队](https://tech.meituan.com/nio.html)
+ [Java 进阶（五）Java I/O 模型从 BIO 到 NIO 和 Reactor 模式](http://www.jasongj.com/java/nio_reactor/)
+ [田小波 - NIO](http://www.tianxiaobo.com/categories/foundation-of-java/NIO/)
