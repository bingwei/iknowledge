# Java 网络编程

## 什么是 socket

Socket 就是网络连接的端点。就像一根网线，一头连到路由器，一头连到电脑。这两端就是 socket。

Socket 本意是"插座"，端口就是插座上的孔。端口不能被其他进程占用，因此 socket 类似于操作某个 IP 地址上的某个端口达到点对点通信的目的, 需要绑定到某个具体的进程中和端口中。

Socket 原本代表 Unix 上的原始套接字 (RawSocket)，用于描述文件的内存镜像。因为 Unix 系统设计哲学是"一切都是文件"，所以后来的网络版的进程间通信就被冠名为"文件描述符" (file descriptor)。很多通讯协议的实现和源代码，都能看到 `Socket* _fd` 这样的变量命名，也正是因为如此。

## 传统方式 (BIO)

### TCP

Client:

```Java
Socket socket = new Socket("127.0.0.1", 4000);
OutputStream os = socket.getOutputStream();
// Write to output stream...
```

Server:

```Java
ServerSocket serverSocket = new ServerSocket(4000);
Socket socket = serverSocket.accept();
InputStream is = socket.getInputStream();
// Read from input stream...
```

### UDP

TODO

## NIO 写法

```Java
Selector selector = Selector.open();
ServerSocketChannel channel = ServerSocketChannel.open();
channel.socket().bind(new InetSocketAddress(4000));
channel.configureBlocking(false);
channel.register(selector, SelectionKey.OP_ACCEPT);

while (true) {
    if (selector.select(TIMEOUT) == 0) {
        continue;
    }
    Iterator<SelectionKey> iter = selector.selectedKeys().iterator();
    while (iter.hasNext()) {
        SelectionKey key = iter.next();
        if (key.isAcceptable()) {
            // ...
        }
        if (key.isReadable()) {
            // ...
        }
        if (key.isWritable() && key.isValid()) {
            // ...
        }
        if (key.isConnectable()) {
            // ...
        }
        iter.remove();
    }
}
```