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

## SQL 运算过程

可以将 SQL 的计算过程类比函数式编程

+ [FROM] 对 relation 做笛卡尔积运算，类比数据源
  + 逗号 —— 笛卡尔积运算
  + natural join —— 自然连接运算
  + join .. using —— 自定义连接运算
+ [WHERE] 选择行，类比 filter
+ [GROUP BY] 分组，类比？？
+ [HAVING] 选择组，类比 filter
+ [SELECT] 选择列，类比 map（普通 SELECT）和 reduce（聚集函数）
+ [DISTINCT] 去重
+ [ORDER BY] 排序
+ [LIMIT] 截取

参考：

+ [Microsoft 文档 - Logical Processing Order of the SELECT statement](https://docs.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql?view=sql-server-2017#logical-processing-order-of-the-select-statement)

从这个计算的过程能看出来变量的作用域。

FROM 子句中的子查询相当于链式函数式编程。

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

## TODO

TODO 数据库设计

TODO 复杂的 SQL

TODO SQL 解析

TODO ORM

TODO NOSQL

TODO SQL/NOSQL 的发展过程