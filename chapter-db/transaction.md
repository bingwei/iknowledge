# 数据库 - 事务 Transaction

## 概念

### 事务的 ACID 性质

+ Atomicity 原子性
  + 事务中包含的所有操作要么全做，要么全不做
+ Consistency 一致性
  + 在事务开始以前，数据库处于一致性的状态，事务结束后，数据库也必须处于一致性状态
+ Isolation 隔离性
  + 事务不受其他并发执行的事务的影响
+ Durability 持久性
  + 事务一旦成功完成，它对数据库的改变必须是永久的，即便是在系统遇到故障的情况下也不会丢失

### 隔离性级别 Isolation Level

#### 脏读、不可重复读、幻读

+ 脏读 _dirty read_ (ref: [MySQL](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_dirty_read), [Wikipedia](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Dirty_reads))
  + 是指读到其他事务更新但未提交的数据（所谓“脏数据”）
    + 因为这些数据随时可能被 rolled back，所以是完全不准确的
  + 也称为不一致读 _inconsisitent read_
  + 只有 read uncommitted 隔离级别才会出现脏读
+ 不可重复读 _non-repeatable read_ (ref: [Wikipedia](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Non-repeatable_reads))
  + 一个事务中两次读了同一行，但是读到的数据不一样
    + 因为在过程中另外一个事务对数据进行了更新并提交了
  + 和脏读的区别是——另一个事务必须提交
  + 不可重复读的本质是因为读锁在读完之后立即释放了
+ 幻读 _phantom problem_ (ref: [MySQL](https://dev.mysql.com/doc/refman/5.7/en/innodb-next-key-locking.html), [Wikipedia](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Phantom_reads))
  + > The so-called phantom problem occurs within a transaction when the same query produces different sets of rows at different times. For example, if a SELECT is executed twice, but returns a row the second time that was not returned the first time, the row is a “phantom” row. 幻读是指在一个事务中，同一个查询会在不同时候返回不同的行数，那些多出的行就好像是出现了“幻象”。
  + 和不可重复读的区别是——行的数量改变了
  + 幻读的本质是因为只用了 [record lock](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-record-locks)，而没有使用 [gap lock](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-gap-locks)，导致其他的事务可以在两行之间的空隙插入新行

#### 四种隔离性级别

SQL:1992 标准定义了四种隔离性级别，但是每种 DBMS 的支持程度和实现方式都不一样。四个隔离性级别从低到高：

+ 未提交读 _read uncommitted_
  + 不上锁，可以认为事务之间完全不隔离
  + 会出现**脏读 (dirty read)** 问题
+ 提交读 _read committed_（一般数据库默认隔离级别）
  + 会使用读锁和写锁对数据进行保护，解决了脏读问题，进行一致性读 (consistent read)
  + 会出现**不可重复读 (non-repeatable read)** 问题
+ 可重复读 _repeatable read_（InnoDB 默认隔离级别）
  + > Consistent reads within the same transaction read the snapshot established by the first read. This means that if you issue several plain (nonlocking) SELECT statements within the same transaction, these SELECT statements are consistent also with respect to each other. 在同一个事务中，多个 SELECT 查询的结果应当是一致的
  + 将读锁和写锁保持到事务结束再释放，解决了不可重复读问题
  + 但是会出现**幻读 (phantom problem)** 问题
+ 序列化读 serializable
  + 最高级别，保证事务之间如同不会交叉一样，每个事务都可以认为只有它自己在操作数据库
  + 使用 gap lock，解决了幻读问题

几个隔离级别是性能和可靠性/一致性等的权衡。一般场景下用 read committed 级别就可以了。InnoDB 的默认隔离级别是 repeatable read。

参考：

+ InnoDB 的事务隔离性级别：[MySQL 5.7 Reference Manual - 14.7.2.1 Transaction Isolation Levels](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html)

#### 对应关系

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :-: | :-: | :-: | :-: |
| Read uncommitted | Y | Y | Y |
| Read committed | | Y | Y |
| Repeatable read | | | Y |
| Serializable | | | |

注意：InnoDB 的 repeatable read 级别一般会使用 gap lock，所以可以避免幻读问题。

## 使用方法

默认设置下，事务是自动提交的，即每执行一条语句都自动提交一次。要禁止自动提交，要么显式使用 `BEGIN; ... COMMIT;`，要么设置 `SET AUTOCOMMIT=0`。

```SQL
BEGIN; % 也可以用 START TRANSACTION;
...
...
COMMIT; % 提交
```

```SQL
BEGIN;
...
...
ROLLBACK;
```

### 设置隔离性级别

```SQL
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

## 事务的实现方式

+ 隔离性 —— 由**并发控制系统**实现（一般是锁）
+ 原子性、持久性 —— 由**恢复系统**实现（一般是 redo log 和 undo log）
  + Redo log 负责崩溃后的恢复，保证持久性
  + Undo log 负责撤销修改，保证原子性
+ 一致性 —— 存在争议
  + > Ensuring consistency for an individual transaction is the responsibility of the application programmer who codes the transaction. 保障一致性是程序员的责任 —— _Database System Concepts_, Section 14.2

### 并发控制系统 Concurrency control system

每个存储引擎实现锁的方式都不同，见 InnoDB。

### 恢复系统 Recovery system

#### ARIES 算法

ARIES (Algorithms for Recovery and Isolation Exploiting Semantics) 一种通用的数据库恢复算法。

ARIES 算法基于 WAL 原则。WAL (Write-Ahead Logging) 是一种用来保证持久性和原子性的方法。日志的内容比实际的修改先写到磁盘。对于 redo log 和 undo log 都是如此。

ARIES 算法的恢复步骤：

+ 根据 redo log 中记录的历史，将系统恢复到 crash 之前的状态
+ 根据 undo log，将未完成的事务撤销

参考：

+ [ARIES数据库恢复算法](https://blog.lotuslab.org/posts/2017-01-02-aries%E6%95%B0%E6%8D%AE%E5%BA%93%E6%81%A2%E5%A4%8D%E7%AE%97%E6%B3%95/)
+ [Write-ahead logging - Wikipedia](https://en.wikipedia.org/wiki/Write-ahead_logging)
+ [Slide - ARIES Recovery Algorithm](https://www.db-book.com/db4/slide-dir/Aries.pdf)

#### Redo Log

Redo log记录某数据块被修改后的值，可以用来恢复未写入data file的已成功事务更新的数据。

+ [MySQL 5.7 Reference Manual  /  The InnoDB Storage Engine  /  InnoDB On-Disk Structures  /  14.6.6 Redo Log](https://dev.mysql.com/doc/refman/5.7/en/innodb-redo-log.html)

#### Undo Log

Undo log 记录某数据被修改前的值，可以用来在事务失败时进行 rollback

#### Redo log 与 undo log 的关系

+ Redo log 一般是写到文件里，而 undo log 是放在一个特殊的段中
+ Redo log 是物理日志，而 undo log 是逻辑日志
  + 物理日志只需要考虑当前页，效率较好
  + 逻辑日志的并发性更好
+ Undo log 也需要持久化保护，因此 undo log 也有对应的 redo log
