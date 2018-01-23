# 显示

## 字体

Ubuntu 14.04 和 16.04 发行版当中都会把楷体设置成 Chrome 浏览器的默认字体，显示效果非常糟糕。安装 Droid Sans 字体，删除难看的楷体。

```Shell
sudo apt-get install fonts-droid
sudo apt-get remove fonts-arphic-ukai fonts-arphic-uming
```

字体文件可以去网上下载，也可以直接从Windows系统的 C:\Windows\Fonts 目录复制，字体放到/usr/share/fonts目录下（可以新建子目录）。

一些字体在Windows中的名字：

+ msyh - 微软雅黑
+ msyhl - 微软雅黑 Light
+ SimSun - 宋体
+ STXiHei - 华文细黑

更新字体缓存：

```
sudo mkfontscale && sudo mkfontdir && sudo fc-cache -fv
```

查看系统里已有的中文字体：

```Shell
fc-list :lang=zh
```

