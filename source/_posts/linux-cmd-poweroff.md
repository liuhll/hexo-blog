---
title: poweroff命令
date: 2018-12-02 21:40:30
categories: "linux"
tags:
- linux命令
---
# 简介
`poweroff命令`用来关闭计算机操作系统并且切断系统电源。

# 语法
```shell
poweroff(选项)
```

# 选项
```shell
-n：关闭操作系统时不执行sync操作；
-w：不真正关闭操作系统，仅在日志文件“/var/log/wtmp”中；
-d：关闭操作系统时，不将操作写入日志文件“/var/log/wtmp”中添加相应的记录；
-f：强制关闭操作系统；
-i：关闭操作系统之前关闭所有的网络接口；
-h：关闭操作系统之前将系统中所有的硬件设置为备用模式。
```

# 实例
如果确认系统中已经没有用户存在且所有数据都已保存，需要立即关闭系统，可以使用`poweroff`命令。

使用poweroff立即关闭系统：

```shell
poweroff
```