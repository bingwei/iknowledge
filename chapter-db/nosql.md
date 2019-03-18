# NoSQL

## 为什么要用 NoSQL

### 性能与一致性

关系型数据库最大特点就是事务的一致性。为了维护一致性，数据的读写性能会较差，无法应付大量的并发读写。

有些业务场景下数据的一致性不是那么重要，可以使用较弱的一致性模型（如最终一致性）

### 存储方式

关系型数据库使用表结构存储数据。这样的存储方式扩展性较差，当系统的功能发生改变时，表结构可能会产生大量的调整。

关系型数据库不适合处理半结构或者非结构化的数据。

## NoSQL 的特点

### 非结构化 denormalized

TODO

### 没有事务 ACID，而是使用最终一致性

针对关系型数据库的 ACID，NoSQL 提出了 BASE：

+ Basically available
+ Soft-state
  + 即使在没有输入的情况下，系统的状态也可能改变
+ Eventually consistent

## 常见的 NoSQL

+ 文档型
  + MongoDB
  + CouchDB
  + DynamoDB
+ 键-值存储
  + Cassandra
  + Redis
+ 图数据库
  + Neo4j
+ 列式存储
  + BigTable
