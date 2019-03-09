# 操作系统 - 文件系统

## 文件

### inode

inode 全名是 _index node_。

inode 中保存文件的各种信息，唯独**不包括文件名**。Linux 使用 inode 号码来识别文件，而不是文件名。

如何从文件名获取 inode 号码？目录文件中存储了文件名和 inode 号码的对应关系。由于目录文件内只有文件名和inode号码，所以如果只有读权限，只能获取文件名，无法获取其他信息，因为其他信息都储存在inode节点中，而读取inode节点内的信息需要目录文件的执行权限（x）。

如何从 inode 号码获取文件名？

参考：

+ _The Linux Programming Interface_, Section 14.4
+ [理解inode - 阮一峰](http://www.ruanyifeng.com/blog/2011/12/inode.html)

### 文件描述符 file descriptor

文件描述符是非负整数，描述**打开的文件**。

+ 0 —— stdin 标准输入
+ 1 —— stdout 标准输出
+ 2 —— stderr 标准错误

Unix 内核会为每个进程维护一个**文件描述符表** (_file descriptor table_)。文件描述符则是这个表的 index，指向**打开文件表** (_open file table_) 中的一项。通过 `dup` / `dup2` 函数，两个文件描述符可以指向同一个打开文件表项。

打开文件表记录了文件的 offset 和 flags（`open()` 的参数），以及指向 _inode table_ 中的一项。一个文件可以被打开多次，有多个打开文件表项，它们的 offset 和 flags 不同，但指向同一个 inode。

![File descriptor table, open file table and inode table](img/file-descriptor-table.png)

（图片来自 _The Linux Programming Interface_）详情参考书的 5.4 节。
