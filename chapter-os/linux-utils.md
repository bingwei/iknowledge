# Linux 命令

## 查看系统信息

### CPU

+ `cat /proc/cpuinfo`
+ `cat /proc/cpuinfo | grep 'cpu cores' | uniq` 查看 CPU 核数
+ `cat /proc/cpuinfo | grep 'processor'` 查看逻辑 CPU 个数

### 内存

+ `free -h` 查看内存与 Swap 的使用量和空余量
+ `cat /proc/meminfo | grep MemTotal` 查看内存总量
+ `cat /proc/meminfo | grep MemFree` 查看内存空余量

### 磁盘

+ `fdisk -l` 查看磁盘分区信息

### 文件系统

+ `df -h` 查看各个文件系统的使用量与挂载点
+ `df -ih` 查看 inode 使用情况（默认是显示 block 使用情况）
+ `du -sh <dir>` 查看目录大小

### 操作系统

+ `uname -s` 查看内核名称
+ `uname -r` 查看内核版本

### 环境变量

+ `env | egrep '^HOME'`

## 进程相关

### uptime

+ `uptime`

简略显示系统的运行时间、用户数、最近负载。`top` 的第一行也会显示这些。

### top

+ `top`

显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。

### ps

+ `ps -ef | grep redis`
+ `ps aux | grep redis`

ps 有两种格式，UNIX 的和 BSD 的。`ps -ef` 是 UNIX 格式的；`ps aux` 是 BSD 格式的，不需要带横线。

+ `-e` Select all processes.
+ `-f` Do full-format listing.
+ `a` Show processes for all users
+ `u` Display user-oriented format
+ `x` Show processes even not attached to terminal

### kill

向进程发送 signal。

+ `kill -9 <pid>` 发送 SIGKILL 信号，杀死进程

## 网络相关

### ifconfig

+ `ifconfig` 查看网络接口信息

### netstat

+ `netstat -nltp | grep 6379`

参数：

+ `-n` 尽量显示数字形式的地址，如 `127.0.0.1` 而不是 `localhost`
+ `-l` 只显示 listening sockets
+ `-a` 显示 listening 和 non-listening sockets（不加参数应该是只显示 non-listening？）
+ `-t` TCP 协议
+ `-u` UDP 协议
+ `-p` 显示 PID 和程序名