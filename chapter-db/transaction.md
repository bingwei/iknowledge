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

### 隔离性级别

SQL 标准定义了四种隔离性级别，但是每种 DBMS 的支持程度和实现方式都不一样。

+ 未提交读 read uncommitted
  + 最低等级
  + 允许读未提交数据
  + 可以认为事务之间完全不隔离
  + 事务 A 开始，接着事务 B 开始，事务 B 更新数据，这时候，A 读取了 B 未提交的数据，这种情况叫做脏读 (dirty read)。此时如果事务 B 遇到错误必须 rollback，那么 A 读取的数据就完全是错的。
+ 提交读 read committed
  + 只允许读已提交数据
  + 事务 A 读数据的过程中，中间事务 B 更新了数据并提交，此时 A 再读，数据发生了改变
  + 不可重复读 non-repeatable reads
+ 可重复读 repeatable read
  + InnoDB 默认隔离级别
  + > if you issue several plain (nonlocking) SELECT statements within the same transaction, these SELECT statements are consistent also with respect to each other. 在同一个事务中，多个 SELECT 查询的结果应当是一致的
  + 只允许读已提交数据，而且在一个事务两次读取一个数据项期间，其他事务不得更新该数据
+ 序列化读 serializable
  + 最高级别，保证事务之间如同不会交叉一样，每个事务都可以认为只有它自己在操作数据库

一般场景下用 read committed 级别就可以了。

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :-: | :-: | :-: | :-: |
| Read uncommitted | Y | Y | Y |
| Read committed | | Y | Y |
| Repeatable read | | | Y |
| Serializable | | | |

注意：InnoDB 的 repeatable read 级别似乎使用 MVCC 机制解决了幻读问题。

一些问题：

+ 脏读 dirty reads
  + 脏数据是指未提交的数据
  + 如果出现脏读，一个事务可能读到另外一个事务中未提交的数据
  + Read uncommitted 隔离级别才会出现脏读
+ 不可重复读 non-repeatable reads
  + 一个事务中两次读到的数据不一样，因为在过程中另外一个事务对数据进行了修改而且提交
  + 和脏读的区别是读到的是已提交的事务
+ 幻读
  + > The so-called phantom problem occurs within a transaction when the same query produces different sets of rows at different times. For example, if a SELECT is executed twice, but returns a row the second time that was not returned the first time, the row is a “phantom” row. 幻读是指在一个事务中，同一个查询会在不同时候返回不同的行数，那些多出的行就好像是出现了“幻象”。
  + 注意和不可重复读区分开，比较难

参考：

+ [关于幻读，可重复读的真实用例是什么？](https://www.zhihu.com/question/47007926)
+ [何为脏读、不可重复读、幻读](http://ifeve.com/db_problem/)
+ [搞懂 不可重复读和幻读](https://segmentfault.com/a/1190000012669504)

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

## 事务的实现方式

+ 隔离性 —— 由**并发控制系统**实现（一般是锁）
+ 原子性、持久性 —— 由**恢复系统**实现（一般是 redo log 和 undo log）
+ 一致性 —— 存在争议
  + > Ensuring consistency for an individual transaction is the responsibility of the application programmer who codes the transaction. 保障一致性是程序员的责任 —— _Database System Concepts_, Section 14.2

### 并发控制系统 Concurrency control system

每个存储引擎实现锁的方式都不同，见 InnoDB。

### 恢复系统 Recovery system

#### Redo Log

Redo log记录某数据块被修改后的值，可以用来恢复未写入data file的已成功事务更新的数据。

#### Undo Log

Undo log 记录某数据被修改前的值，可以用来在事务失败时进行 rollback

#### Redo log 与 undo log 的关系

+ Redo log 一般是写到文件里，而 undo log 是放在一个特殊的段中
+ Redo log 是物理日志，而 undo log 是逻辑日志
+ Undo log 也需要持久化保护，因此 undo log 也有对应的 redo log
