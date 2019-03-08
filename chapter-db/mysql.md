# MySQL 

## InnoDB

参考：

+ [MySQL documentation - Chapter 14 The InnoDB Storage Engine](https://dev.mysql.com/doc/refman/5.5/en/innodb-storage-engine.html)
+ [『浅入浅出』MySQL 和 InnoDB](https://draveness.me/mysql-innodb)

## 基本使用

登录

```Bash
mysql -u username -p
```

```Bash
mysql -u username -p db_name
```

```Bash
mysql -u username -p db_Name < schema.sql
```

创建管理员用户

```SQL
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
```

创建数据库

```SQL
CREATE DATABASE db_name CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

参考：

+ [MySQL入门——看这一篇就够了](https://zhuanlan.zhihu.com/p/22893582)
