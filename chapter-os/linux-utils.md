# Linux 命令

## 进程相关

### ps

`ps -aux | grep redis`

ps 有两种格式，UNIX 的和 BSD 的。`ps -ef` 是 UNIX 格式的；`ps aux` 是 BSD 格式的，不需要带横线。

+ `-e` Select all processes.
+ `-f` Do full-format listing.
+ `a` Show processes for all users
+ `u` Display user-oriented format
+ `x` Show processes even not attached to terminal

### netstat

`netstat -nlt | grep 6379`

+ `-n`, `--numeric` Show numerical addresses instead of trying to determine symbolic host, port or user names.
+ `-l`, `--listening` Show only listening sockets
+ `-t`, `--tcp` TCP protocol