# Redis

Redis是一个开源的内存中的数据结构存储系统，它可以用作：

+ 数据库
  + NoSQL, Key-value
+ 缓存
  + 取代 memcached
+ 消息中间件

[Redis常见的应用场景解析](https://zhuanlan.zhihu.com/p/29665317)

Redis 使用**单进程单线程**，而 memcached 使用**单进程多线程**。Redis 的 I/O 模型使用多路复用 I/O multiplexing。

+ [为什么说Redis是单线程的以及Redis为什么这么快！](https://blog.csdn.net/xlgen157387/article/details/79470556)
+ [Redis 和 I/O 多路复用](https://draveness.me/redis-io-multiplexing)

## 数据结构

四大基础集合数据结构：

+ SET
+ LIST
+ SORTED SET
+ HASH

集合不能嵌套，也就是说集合的成员必须是 bytestring。

## 参考资料

+ [Redis tutorial](https://static.simonwillison.net/static/2010/redis-tutorial/)