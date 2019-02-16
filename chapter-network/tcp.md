# TCP 协议

## 可靠连接

TCP 是 _全双工 (full deplex)_ 的，允许数据在两个方向上同时进行传输。

TCP 的重点在于**可靠**的数据传输。IP 和 UDP 不进行任何的 retry；Ethernet 会做一些 retry，但是不会坚持。TCP 则会坚持进行重传 (retransmission)。

IP 层可能会出现的问题：

+ Packet bit error
+ Packet drop (丢包)
+ Packet reordering (包乱序)
+ Packet duplication

其中 packet bit error 和 packet drop 的问题用 acknowledgement 来解决；packet reordering 和 packet duplication 的问题用 sequence number 来解决。

### Sequence number 序列号

TCP 建立可靠连接的重要基础是 _序列号 (sequence number)_。序号的作用：

+ 保证数据的顺序
  + 乱序到达的 segment 会根据序号进行重排
+ 当连接断开并重连后，可以区分新旧连接中的 segment
  + 每个连接都有一个独一无二的 _初始序列号 (initial sequence number, ISN)_

## TCP header

![TCP header](img/tcp-header.jpg)

![TCP header footnote](img/tcp-header-2.jpg)

+ Sequence number (seq, 序号)
  + 以字节为单位
  + 建立连接时称为 _SYN seq_，断开连接时称为 _FIN seq_
+ Acknowledgement number (ACK, 确认号)
  + 确认收到了该序号之前 (exclusive) 的所有字节
  + 确认号=期望收到的下一个数据字节的编号
+ Window: 滑动窗口，解决流量控制问题
+ TCP flags: 代表包的类型，用来操作 TCP 的状态机
  + ACK 确认信号
    + 仅当 ACK=1 时，确认号才有效
    + 连接建立后，所有 segment 都必须有 ACK=1
  + SYN 同步信号
    + 在建立连接时用来同步序号（此时序号称为 _SYN seq_）
    + SYN=1 的 segment 不能携带数据，但要消耗掉一个序号
  + FIN 终止信号
    + FIN=1 表示要求释放连接（此时序号称为 _FIN seq_）

## TCP 状态机

![TCP state machine](img/tcp-state-machine.png)

## 连接过程

![TCP open/close](img/tcp-open-close.jpg)

### 建立连接：三步握手 (Three way handshake)

建立连接的重点，是通信双方告诉对方自己生成的**初始序列号 ISN**。也就是进行序列号的**同步** (SYN=1)。

+ Client 发送 SYN seq=x
+ Server 发送 ACK seq=x+1, SYN seq=y
+ Client 发送 ACK seq=y+1

这三次通信是必须的，因为 TCP 需要双向连接都是可靠的，双方都必须知道对方的 ISN。（如果只需要单向发送数据，理论上可以只要两次握手。）

```
      TCP A                                                TCP B

  1.  CLOSED                                               LISTEN

  2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

  3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

  4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK>       --> ESTABLISHED

  5.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED

          Basic 3-Way Handshake for Connection Synchronization
```

参考：

+ [TCP 为什么是三次握手，而不是两次或四次？- 知乎](https://www.zhihu.com/question/24853633)

### 断开连接：四次挥手

其实你仔细看是 2 次，因为 TCP 是全双工的，所以，发送方和接收方都需要 Fin 和 Ack。只不过，有一方是被动的，所以看上去就成了所谓的 4 次挥手。如果两边同时断连接，那就会就进入到 CLOSING 状态，然后到达 TIME_WAIT 状态。

## 流量控制

### 滑动窗口

Sender 和 receiver 都有一个滑动窗口。