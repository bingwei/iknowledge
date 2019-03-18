# 数据库

## 概念

+ 关系模型 _relation model_
  + 表 _table_
  + 关系 _relation_ —— 表
  + 元组 _tuple_ —— 表中一行
  + 属性 _attribute_ —— 表中一列
+ 数据库模式 _database schema_ —— 数据库的逻辑设计
+ 数据库实例 _database instance_ —— 数据库某时刻的数据快照
+ 键（码） _key_
  + _Superkey_ —— 保证不重复的属性集合
  + （通常意义上教材中说的**键**）候选键 _candidate key_ —— superkey 中最小的集合
  + 主键 _primary key_ —— 用户选定的候选键

“关系模式”和“关系”的区别，类似于面向对象程序设计中”类“与”对象“的区别。”关系“是”关系模式“的一个实例，你可以把”关系”理解为一张带数据的表，而“关系模式”是这张数据表的表结构。

## SQL

### SQL 语句分类

+ _Data definition language_ (DDL)
  + Schema 相关？
  + CREATE, DROP, ALTER, TRUNCATE
+ _Data control language_ (DCL)
  + Control access，权限相关
  + GRANT, INVOKE
+ _Data manipulation language_ (DML)
  + 修改（写）数据库实例
  + INSERT, DELETE, UPDATE
+ _Data query language_ (DQL)
  + 查询（读）数据库实例
  + SELECT

### JOIN

+ INNER JOIN = JOIN
+ LEFT JOIN = LEFT OUTER JOIN
+ RIGHT JOIN = RIGHT OUTER JOIN
+ FULL JOIN = FULL OUTER JOIN

![SQL joins](img/sql-joins.png)

无论何种 join，一定要加一个 `ON A.key = B.key`

参考：

+ [Join (SQL) - Wikipedia](https://en.wikipedia.org/wiki/Join_(SQL))
+ [图解SQL的JOIN - CoolShell](https://coolshell.cn/articles/3463.html)

## SQL 运算过程

可以将 SQL 的计算过程类比函数式编程

+ [FROM] 对 relation 做笛卡尔积运算，类比数据源
  + 逗号（相当于 CROSS JOIN） —— 笛卡尔积运算
  + JOIN —— 连接运算
  + ON —— 选择行
+ [WHERE] 选择行，类比 filter
+ [GROUP BY] 分组，类比？？
+ [HAVING] 选择组，类比 filter
+ [SELECT] 选择列，类比 map（普通 SELECT）和 reduce（聚集函数）
+ [DISTINCT] 去重
+ [ORDER BY] 排序
+ [LIMIT] 截取

### JOIN

数据库的实现一般不会直接做笛卡尔积，因为太慢。两种常见的实现方式：

+ [Hash join](https://en.wikipedia.org/wiki/Hash_join)
  + 只能用于比较 `=` 的 join
+ [Sort-merge join](https://en.wikipedia.org/wiki/Sort-merge_join)
  + 根据 join 的属性排序，参考 external sort

参考：

+ [Microsoft 文档 - Logical Processing Order of the SELECT statement](https://docs.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql?view=sql-server-2017#logical-processing-order-of-the-select-statement)

从这个计算的过程能看出来变量的作用域。

FROM 子句中的子查询相当于链式函数式编程。

<<<<<<< HEAD
=======
SQL 语句分类

+ _Data definition language_ (DDL)
  + Schema 相关？
  + CREATE, DROP, ALTER, TRUNCATE
+ _Data control language_ (DCL)
  + Control access，权限相关
  + GRANT, INVOKE
+ _Data manipulation language_ (DML)
  + 修改（写）数据库实例
  + INSERT, DELETE, UPDATE
+ _Data query language_ (DQL)
  + 查询（读）数据库实例
  + SELECT

## 存储过程 Stored Procedure

存储过程是由一些 SQL 语句组成的代码块，相当于一个函数。

存储过程的优点：

+ 是编译过的代码块，执行效率高
+ 降低网络中通信
+ 使用参数，防止 SQL 注入攻击

存储过程的缺点：

+ 可移植性差，不同的 DBMS 支持的语法可能不一样
+ 系统难以维护

阿里巴巴 Java 开发手册里禁止使用存储过程。

>>>>>>> 43a7e831140d137a1de531a73cb81df4274a529c
## 范式 Normal form

范式就是**一张数据表的表结构所符合的某种设计标准的级别**。符合高一级范式的设计，必定符合低一级范式。

范式是从**语义**上分析的。

一般的数据库设计需要符合 3NF 或 BCNF。

概念：

+ _Partial dependency_ 部分函数依赖
+ _Transitive dependency_ 传递函数依赖

常见范式：

+ 1NF
  + 每个属性都不可再分，即每个单元格都只有一个值
  + 1NF 是所有关系型数据库的最基本要求
+ 2NF
  + 在 1NF 的基础上，消除**非主属性**对于**候选键**的**部分函数依赖**
  + 也就是说，所有的值都应当依赖整个候选键，而不是依赖部分候选键
+ 3NF
  + 在 2NF 的基础上，消除**非主属性**对于**候选键**的**传递函数依赖**
  + 也就是说，所有的依赖都是对候选键的依赖，没有对非候选键的依赖
+ BCNF
  + 在 3NF 的基础上，消除**主属性**对于**候选键**的**部分&传递函数依赖**
  + 也就是说，所有的依赖都是对 superkey 的依赖

规范化过程就是不断进行**模式分解**的过程（即对表进行拆分）。

参考：

+ [Database normalization - Wikipedia](https://en.wikipedia.org/wiki/Database_normalization)
+ [如何解释关系数据库的第一第二第三范式？ - 知乎](https://www.zhihu.com/question/24696366)

## 事务

### 事务的 ACID 性质

+ Atomicity 原子性
  + 事务中包含的所有操作要么全做，要么全不做
+ Consistency 一致性
  + 在事务开始以前，数据库处于一致性的状态，事务结束后，数据库也必须处于一致性状态
+ Isolation 隔离性
  + 事务不受其他并发执行的事务的影响
+ Durability 持久性
  + 事务一旦成功完成，它对数据库的改变必须是永久的，即便是在系统遇到故障的情况下也不会丢失

实现方式：

+ 隔离性 —— 由**并发控制机制**（一般是锁）实现
+ 原子性、持久性 —— 由 **REDO 日志**实现
+ 一致性 —— 由 **UNDO 日志**实现

### 锁

每个存储引擎实现锁的方式都不同，见 InnoDB。

### 隔离性

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

## TODO

TODO 数据库设计

TODO 复杂的 SQL

TODO SQL 解析

TODO ORM

TODO NOSQL

TODO SQL/NOSQL 的发展过程