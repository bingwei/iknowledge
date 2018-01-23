# 网络

## Privoxy 将SOCKS5代理转换为HTTP代理

```Bash
sudo vim /etc/privoxy/config
```

插入一行
```
forward-socks5    /    127.0.0.1:1080     .
```

注意：不要忘记最后那一个点！

```Bash
sudo service privoxy restart
```
