# 架构设计 / 系统设计

## 一些概念

### Load balancer (负载均衡器)

负载 (load) 即用户发来的请求

### Reverse proxy (反向代理)

TODO

### Service discovery

TODO

### Sharding (分片)

### Consistent hashing (一致性哈希)

背景：分片 (sharding)，根据 hash(key) 决定将数据放在哪个 shard 上。

+ 目的：shard 的数目能够动态的增加/减少
+ Hash 的值域为 32 位整型 (0 <= h < 2^32)，组织成一个圆环
+ 每个 shard 确定其在圆环上的位置
+ hash(key) 顺时针走，遇到的第一个 shard 即为目标 shard
+ 可以引入虚拟节点，避免单个 shard 负载过重

Memcached, Dynamo, Cassandra 等都使用到了一致性哈希

## 一些工具

+ Key-value 缓存：memcached, Redis
+ 消息队列：RabbitMQ, ActiveMQ, Kafka
+ 负载均衡器 / 反向代理：nginx, HAProxy
+ 多服务器部署：Capistrano
+ NoSQL
  + Key-value: Memcached, Redis
  + 文档型：MongoDB, DynamoDB, ElasticSearch
  + 列型：HBase, Cassandra
+ 服务发现：Consul
+ 运维工具：Docker, Kubernetes
+ 暂时无分类：Etcd, Zookeeper