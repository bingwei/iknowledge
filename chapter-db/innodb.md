# InnoDB

## InnoDB 与 MyISAM 的区别

+ InnoDB 支持事务，MyISAM 不支持
+ InnoDB 支持外键，MyISAM 不支持
+ InnoDB 是聚集索引，数据和索引放在一起，必须要有主键，辅助索引需要先查询到主键，再通过主键查询到数据。而 MyISAM 是非聚集索引，数据和索引分离，索引保存的是数据文件的指针。
+ InnoDB 不保存表的具体行数，执行 `select count(*) from table` 时需要全表扫描。而 MyISAM 用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快
+ MyISAM 更适合读操作很多的情况，而 InnoDB 适合读写操作都很多的情况
+ MyISAM 的锁是表锁，而 InnoDB 支持行级锁

## InnoDB 数据存储方式

+ .frm 文件
  + 存储表结构定义（包括视图定义）
  + 所有存储引擎都一样，不仅仅是 InnoDB
  + 参考：[11.1 MySQL .frm File Format](https://dev.mysql.com/doc/internals/en/frm-file-format.html)
+ .idb 文件
  + 存储数据（包括记录和索引）

行格式（记录存储方式）：

+ Antelope 文件格式
  + Compact
  + Redundant
+ Barracuda 文件格式
  + Compressed
  + Dynamic

Compact 是基础，其他的几个格式都是和 compact 相比稍微有点变化。

存储大对象（包括很长的 VARCHAR 或者 BLOB）时，行数据会溢出：

+ Compact 或者 Redundant 格式 —— 前 768 个字节存储在行记录中，后面会通过偏移量指向溢出页
+ Compressed 或者 Dynamic 格式 —— 行记录中只存储 20 个字节的指针，全部数据都放在溢出页中

### 表空间

+ 段 segment 是逻辑概念
  + Leaf 段，non-leaf 段
+ 区 extent 是物理概念
  + 碎片区（直属表空间）
  + 属于段的区

### On-Disk Structures

InnoDB 的 index page 默认大小是 16KB。

对于聚集索引，InnoDB 会保留 1/16 的页空间。也就是说每页的空间在 1/2 ~ 15/16 之间。

参考：

+ [『浅入浅出』MySQL 和 InnoDB](https://draveness.me/mysql-innodb)

## 锁

两种行级锁：

+ 共享锁，读锁 (S)
+ 排他锁，写锁 (X)

两种意向锁 (intention lock)，是表级别的：

+ 意向共享锁 (IS)
  + `SELECT ... LOCK IN SHARE MODE` 会设置该锁
+ 意向排他锁 (IX)
  + `SELECT ... FOR UPDATE` 会设置该锁

锁的兼容性：

+ IS 与 IS，IX，S 兼容
+ IX 与 IS，IX 兼容
+ S 与 IS，S 兼容

### Next-key lock

InnoDB 默认工作在 repeatable read 级别下，会使用 next-key locks 来防止幻读问题。

### 多版本并发控制 Multi Version Concurrency Control (MVCC)



## InnoDB 特色

### Insert buffer

InnoDB 对于自增主键的插入较快，但是其他的辅助索引可能就较慢。InnoDB 有 insert buffer 功能，可以加速索引的插入和更新操作。Insert buffer 可以缓存多个操作，将操作合并。

注意 insert buffer 只能用于**非唯一的辅助索引**，因为唯一的索引在插入之前必须判断是否存在。因此，聚集索引不可以使用 insert buffer。

## 参考资料

+ [MySQL documentation - Chapter 14 The InnoDB Storage Engine](https://dev.mysql.com/doc/refman/5.5/en/innodb-storage-engine.html)