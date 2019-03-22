FTP vs SFTP：FTP 协议太老所以明文传输用户名和密码？

一个计算机可以有多 CPU（双 CPU），一个 CPU 可以有多核 (dual-core, quad-core)，所以现在计算机可以真正并行地做任务，以前只是用 进程/线程 调度造成并行的假象

垂直扩展，可能到达现有技术的上限，而且买一个性能顶尖的机器也很贵；水平扩展，则可以用不那么贵的机器，甚至是一些便宜的旧机器。

每次发 HTTP 请求只可能发往一个 IP 地址，如何利用多个服务器呢？

方法一，让 DNS 服务器以 round robin 模式工作

缺点：

+ 某些服务器可能不巧负载太高
+ 丢失掉了缓存
  + （没太听懂）解决方法：DNS 服务器有缓存机制，对同一用户会返回同样的 IP 地址（在一段时间内，直到 TTL 到期）

方法二，使用负载均衡器 LB

DNS 返回负载均衡器的地址，用户的 HTTP 请求发给负载均衡器，负载均衡器做反向代理的功能

负载均衡器根据一些 heuristic 的标准，将用户请求平均分给各个服务器。负责均衡的几种办法：

+ Round robin
+ 划分为 HTML 服务器、IMG 服务器、VIDEO 服务器等
+ 查询每个服务器的负载，分配给负载最小的

负载均衡机制会打破 session 模型。Session 通常是用文本文件保存的，即保存在每个服务器的硬盘上。负载均衡器如果把来自同一用户的请求发给不同的服务器，用户的 session 会丢失，如购物车内容会丢失。解决方法：让负载均衡器保存 session；增加 session 服务器（本质上是 file 服务器）（为了避免单点失效问题，可以使用 RAID）。

负载均衡器举例

软件：ELB, HAProxy, LVS
硬件：Barracuda, Cisco, Citrix, F5

Sticky session: 用户的 session 能够被保存下来，即使是不同的服务器在处理。Cookie 可以解决一部分问题（将服务器的标志存储在 cookie 中，不需要是服务器 IP，可能只是一个随机数，防止服务器 IP 暴露）

Caching

+ 缓存动态生成的 HTML
+ MySQL query cache 
  + in `my.cfg`, add `query_cache_type = 1`
+ 分布式缓存 memcached
  + 数据库还是需要从文件读，这里直接缓存在内存里
  
Replication: master-slave

+ Read 请求通过负载均衡器，请求任意一个 slave 即可
+ Write 请求发给 master，将操作复制到每一个 slave

Replication: master-master

master 间相互复制，一个 master 坏了还能有一个 master

避免单点失效：使用两个负载均衡器

+ active-active: 两个都能接受请求
+ active-passive: active 挂的时候，passive 发现 heartbeat 没有了，自己变成 active

Partitioning

+ 数据库常见
+ 一半的 slave 存 A-M，另一半存 N-Z

High availability (HA)

+ 尽量避免单点失效 single-point failure
+ 异地容灾

Firewall: security

只允许部分端口

+ 80/443 for HTTP
+ 3306 for MySQL

