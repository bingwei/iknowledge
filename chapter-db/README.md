# 数据库

## 概念

+ 关系模型 _relation model_
  + 表 _table_
  + 关系 _relation_ —— 表
  + 元组 _tuple_ —— 表中一行
  + 属性 _attribute_ —— 表中一列
+ 数据库模式 _database schema_ —— 数据库的逻辑设计
+ 数据库实例 _database instance_ —— 数据库某时刻的数据快照
+ 键 _key_
  + _Superkey_ —— 保证不重复的属性集合
  + 候选键 _candidate key_ —— superkey 中最小的集合
  + 主键 _primary key_ —— 用户选定的候选键

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

TODO 数据库设计

TODO 复杂的 SQL

TODO SQL 解析

TODO ORM

TODO NOSQL

TODO SQL/NOSQL 的发展过程