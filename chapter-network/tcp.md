# TCP 协议

TCP 协议面向数据流 (_data stream_)，将数据分块，加上 TCP header 成为一个 TCP 包 (_segment_)。

+ _MTU (maximum transmission unit)_ 最大传输单元
  + 以太网，MTU = 1500
+ _MSS (maximum segment size)_ 最大分段大小
  + MSS = MTU - 20 (IP header) - 20 (TCP header) = 1460

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

ISN 会和一个假的时钟绑定，每 4 us 加一，大约 4.5 小时之后，ISN 会绕回原先的值。

## TCP header

![TCP header](img/tcp-header.jpg)

![TCP header footnote](img/tcp-header-2.jpg)

+ Sequence number (seq, 序号)
  + 以字节为单位
  + SYN=1 时，表示 _initial sequence number_
  + 建立连接时称为 _SYN seq_，断开连接时称为 _FIN seq_
+ Acknowledgement number (ACK, 确认号)
  + 确认号=期望收到的下一个数据字节的编号
  + 确认收到了该序号之前 (exclusive) 的所有字节
+ Window: 滑动窗口，解决流量控制问题
+ TCP flags: 代表包的类型，用来操作 TCP 的状态机
  + ACK 确认信号
    + 当 ACK=1 时，acknowledgement number 有效
    + 连接建立后，所有 segment 都必须有 ACK=1
  + SYN 同步信号
    + 在建立连接时用来同步序号（此时序号称为 _SYN seq_）
    + SYN=1 的 segment 不能携带数据，但要消耗掉一个序号
  + FIN 终止信号
    + 表示是发送方的最后一个 segment
    + FIN=1 表示要求释放连接（此时序号称为 _FIN seq_）
    + FIN=1 的 segment 不能携带数据，但要消耗掉一个序号

## 连接过程

TCP 协议可以分为三个阶段：

+ 连接建立 (_connection establishment_)
+ 数据传输 (_data transfer_)
+ 连接断开 (_connection termination_)

### TCP 状态机

TCP 所谓的“连接”，实际上是通讯的双方维护一个**连接状态**。

![TCP state machine](img/tcp-state-machine.png)

+ SYN-SENT
  + 已经发送了 SYN
  + 等待对方也发送 SYN 和回复 ACK
+ SYN-RECEIVED
  + 已经收到了 SYN，自己也已经发出了 SYN（先后关系不一定）
  + 等待对方回复 ACK
+ ESTABLISHED
  + 自己发出的 SYN 收到了 ACK
  + 进入数据传输阶段
+ FIN-WAIT-1
  + 已经发送了 FIN
  + 等待对方回复 ACK
+ FIN-WAIT-2
  + 自己发出的 FIN 收到了 ACK，自己这一半的连接断开
  + 等待对方也发送 FIN
+ TIME-WAIT
  + 自己这一半的连接已经断开，也已经收到了 FIN
  + 等待超时之后关闭
+ CLOSE-WAIT
  + 已经收到了 FIN
  + 等待自己的程序结束之后发送 FIN
+ CLOSING
  + 已经收到了 FIN，自己也已经发出了 FIN
  + 等待对方回复 ACK
+ LAST-ACK
  + 已经收到了 FIN，自己也已经发出了 FIN
  + 等待对方回复 ACK

### 建立连接：三次握手 (three-way handshake)

建立连接的重点，是通信双方告诉对方自己生成的**初始序列号 ISN**。也就是进行序列号的**同步** (SYN=1)。

+ Client 发送 SYN (seq=x)
+ Server 发送 ACK (seq=x+1), SYN (seq=y)
+ Client 发送 ACK (seq=y+1)

其中第 1,2 步确定了 client->server 方向的 ISN；第 2,3 步确定了 server->client 方向的 ISN。这样就建立了一个全双工的连接。由此可见这三次通信是必须的。（如果只需要单向发送数据，理论上可以只要两次握手。）

状态机状态变化：

+ 一方 passive open，一方 active open
  + LISTEN -> SYN-RECEIVED -> ESTABLISHED
  + SYN-SENT -> ESTABLISHED
+ 双方同时 active open
  + SYN-SENT -> SYN-RECEIVED -> ESTABLISHED

如果 client 发送 SYN 之后掉线，server 会重传 SYN+ACK。Linux 下，默认重试 5 次，时间间隔分别为 1s, 2s, 4s, 8s, 16s，最后一次之后还要等待 32s 超时，总共等待 63s。基于这个特定可以制造 _SYN flood 攻击_。

参考：

+ [TCP 为什么是三次握手，而不是两次或四次？- 知乎](https://www.zhihu.com/question/24853633)

### 断开连接：四次挥手 (four-way handshake)

四次挥手实际上由 2*2 组成。当一方想要停止自己这一半的连接时，就发送 FIN，然后另一方需要回复 ACK。（连接可以是“半开”状态，此时只有一方能发送数据，另一方只能接收数据。）因为两方都需要发送 FIN 和接收 ACK，所以一共是四次挥手。

如果 server 在收到 FIN 的时候，自己也决定不传输数据了。就可以发送 FIN+ACK，将四次挥手合并为三次。

状态机状态变化：

+ 一方先关闭，另一方后关闭
  + FIN-WAIT-1 -> FIN-WAIT-2 -> TIME-WAIT -> CLOSED
  + CLOSE-WAIT -> LAST-ACK -> CLOSED
+ 一方先关闭，另一方马上关闭
  + FIN-WAIT-1 -> FIN-WAIT-2 -> TIME-WAIT -> CLOSED
  + (CLOSE-WAIT ->) LAST-ACK -> CLOSED
+ 双方同时关闭
  + FIN-WAIT-1 -> CLOSING -> TIME-WAIT -> CLOSED

为什么 TIME-WAIT 要等待一个超时时间？

+ 如果自己发出的 ACK 丢失，让对方还有机会重发 FIN
+ 有足够的时间让这个连接不会和后面的连接混淆

![TCP open/close](img/tcp-open-close.jpg)

## 数据传输

### 数据重传

ACK 只会确认最后一个连续收到的包，表示 acknowledgement number 之前的字节都已经收到了。

+ _Fast retransmit_ 快速重传
  + 三次 ACK 就重传，不等到 timeout
+ _Selective acknowledgement_ (SACK)
  + ACK 除了返回连续收到的字节，还用 SACK 返回不连续的字节
  + 发送方可以知道哪些数据需要重传
+ _Duplicate SACK_ (D-SACK)
  + 如果 SACK 的字节范围被 ACK 覆盖，则是 D-SACK，表示这部分字节重复收到了
  + 发送方可以知道
    + 是数据丢包了，还是 ACK 丢包了
    + 网络上是否有先发后到的包（即 reordering）

### 流量控制

在数据重传中，_RTO (retransmission timeout)_ 的设置非常重要。过长过短都不合适，需要根据网络的情况动态设置。

_RTT (round trip time)_，是指一个数据包从发出去到回来的时间（来回时间）。

### 滑动窗口

TCP 需要知道网络速度和数据处理速度，防止引起网络拥塞，导致丢包。

接收方使用 Window 字段告诉发送方自己还可以接收数据的大小，发送方以此控制发送的数据量，防止接收方来不及处理。

Sender 和 receiver 都有一个滑动窗口。下图是发送方的滑动窗口。

![Window of sender](img/window-sender.png)

### 拥塞控制

TCP 拥塞控制有多个实现版本：

+ TCP Tahoe
  + 最简单的版本，没有 fast recovery
+ TCP Reno
  + 在三次 duplicated ACKs 之后，会进入 fast recovery
+ TCP NewReno
  + 在 fast recovery 阶段呆的时间更长

一些概念：

+ _cwnd (congestion window_)
+ _ssthresh (slow start threshold)_
  + 一般设置为 65535B

#### 慢启动 slow start

+ 初始化 cwnd=1，表示可以传一个 MSS 大小的数据
+ 每当收到一个 ACK，cwnd 加一
+ 每当过了一个 RTT，cwnd 翻倍
+ 当 cwnd >= ssthresh 时，进入拥塞避免算法

#### 拥塞避免 congestion avoidance

+ 每当收到一个 ACK，cwnd = cwnd + 1/cwnd
+ 每当过了一个 RTT，cwnd 加一

#### 拥塞发生

+ 如果**超时重传**（较糟糕的情况）
  + ssthresh = cwnd / 2
  + cwnd 重置为 1，重新开始慢启动
+ 如果收到三次 ACK，进入**快速重传**
  + cwnd = cwnd / 2
  + ssthresh = cwnd
  + 进入快速恢复算法

注意：上述快速重传时的算法是 TCP Reno 的实现。TCP Tahoe 的实现也按照超时重传来处理。

#### 快速恢复 fast recovery

+ cwnd = ssthresh + 3
+ 重传 duplicated ACKs 指定的数据
+ 如果再收到 duplicated ACK，cwnd 加一
+ 如果收到了新的 ACK，cwnd = ssthresh，进入拥塞避免算法

## 参考资料

+ TCP 的那些事儿 [（上）](https://coolshell.cn/articles/11564.html) [（下）](https://coolshell.cn/articles/11609.html)
+ [从 TCP 三次握手说起：浅析 TCP 协议中的疑难杂症 (1)](https://cloud.tencent.com/developer/article/1004327)