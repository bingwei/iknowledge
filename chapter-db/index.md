# 数据库 - 索引 Index

## 概念

### 聚集索引与非聚集索引

+ 聚集索引 clustering index，主索引 primary index
  + 索引的顺序定义了表中的物理存储顺序
+ 非聚集索引 nonclustering index，辅助索引 secondary index

## 使用方法

### 建立索引

```SQL
CREATE INDEX index_name ON table_name (column_name);
```

### 全文索引 Full-text index

MySQL 5.6 开始，InnoDB 支持全文索引。

```SQL
CREATE TABLE opening_lines (
    id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
    opening_line TEXT(500),
    author VARCHAR(200),
    title VARCHAR(200),
    FULLTEXT idx (opening_line)
) ENGINE=InnoDB;
```

```SQL
SELECT * FROM table_name WHERE MATCH(column_name) AGAINST('聪')
```

全文索引是以词为基础的。

### 索引最佳实践

#### InnoDB 主键的设计原则

> If your table is big and important, but does not have an obvious column or set of columns to use as a primary key, you might create a separate column with auto-increment values to use as the primary key. 如果表中没有明显的一列是主键，那么添加一个自增列作为主键。

自增列可以保证顺序插入，插入速度快。否则，如果使用随机的 UUID，插入速度就较慢。

主键不应过大，因为每个辅助索引里都会存储主键的值，使得辅助索引占用大量空间。

#### 建立索引的原则

索引加快 SELECT 的速度，但减慢 INSERT，UPDATE 和 DELETE 的速度。索引需要耗费大量空间。所以索引的建立需要进行权衡。

+ 一般需要建立索引的情况：
  + 主键
  + 外键
  + 需要进行范围查询的列
  + 经常出现在关键字 ORDER BY, GROUP BY, DISTINCT 后面的列
+ 一般不要建立索引的情况：
  + 重复值比较多的列
  + TEXT, BLOB 类型的列
  + 经常更新的列

继续阅读：[8.3 Optimization and Indexes](https://dev.mysql.com/doc/refman/5.6/en/optimization-indexes.html)

## 索引的实现

### B+ 树

TODO

### InnoDB 的索引实现

InnoDB 是聚集索引，数据存在 B+ 树的叶结点，索引存在 B+ 树的非叶结点。

InnoDB 选择聚集索引的顺序：

+ 如果有主键，那么主键作为聚集索引
+ 如果没有主键，第一个**唯一 (unique) 非空 (not null) 索引**作为聚集索引
+ 如果没有唯一非空索引，那么 InnoDB 自动生成一个隐藏的主键（6字节，自增）作为聚集索引

InnoDB 中，辅助索引存储的是主键（聚集索引）的值，并不存储记录的物理地址。

聚集索引的优点：

+ 范围查找快速
+ 已经是排好序的

聚集索引的主要缺点是插入的开销很大。
