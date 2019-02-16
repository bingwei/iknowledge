# 计算机网络

## TCP/IP 五层协议，OSI 七层协议

五层协议：

+ (5) 应用层 (_Application layer_)
  + _Data_
+ (4) 传输层 (_Transport layer_)
  + UDP: 报文 (_datagram_)
  + TCP: 流 (_data stream_), 分段/包 (_segment_)
+ (3) 网络层 (_Network layer_)
  + IP: 数据报 (_packet_)
+ (2) 链路层 (_Link layer_)
  + Ethernet, Wi-Fi: 帧 (_frame_)
+ (1) 物理层

OSI 七层协议将 **(5) 应用层** 分割成了 **(5) 会话层、(6) 表示层、(7) 应用层**。

![UDP encapsulation](img/udp-encap.png)

### Application layer 应用层

定义应用进程（进程：主机中正在运行的程序）间的通信和交互的规则。

### Transport layer 传输层

| UDP | TCP |
| :-: | :-: |
| 无连接 | 有连接 |
| 面向报文 (datagram) | 面向字节流 (stream) |
| 不保证可靠交付 | 可靠交付：无差错、不丢失、不重复、按序到达 |
| 一对一、一对多、多对多 | 一对一全双工 |
| 8 字节 header | 20 字节 header + 0~40 字节 option |

### Network layer 网络层

### Data link layer 数据链路层



TODO DNS

## 参考资料

+ 各个协议在 OSI 七层协议中的位置及相互关系：img/osi-7-layer-protocols.png （图片太大不方便放进来）