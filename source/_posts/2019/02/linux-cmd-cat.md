---
title: cat命令
date: 2019-02-16 21:03:00
categories: "linux"
tags:
- linux命令
---

# 简介
`cat`命令连接文件并打印到标准输出设备上，`cat`经常用来显示文件的内容。

> **Notes:**
> - 当文件较大时，文本在屏幕上迅速闪过（滚屏），用户往往看不清所显示的内容。因此，一般用`more`等命令分屏显示。为了控制滚屏，可以按`Ctrl+S`键，停止滚屏；按`Ctrl+Q`键可以恢复滚屏。按`Ctrl+C`（中断）键可以终止该命令的执行，并且返回Shell提示符状态。

# 语法
```shell
cat(选项)(参数)
```

# 选项 
```shell
-n或-number：有1开始对所有输出的行数编号；
-b或--number-nonblank：和-n相似，只不过对于空白行不编号；
-s或--squeeze-blank：当遇到有连续两行以上的空白行，就代换为一行的空白行；
-A：显示不可打印字符，行尾显示“$”；
-e：等价于"-vE"选项；
-t：等价于"-vT"选项；
```
# 参数
文件列表：指定要连接的文件列表。

# 参数
设ml和m2是当前目录下的两个文件
```shell
cat m1  #（在屏幕上显示文件ml的内容）
cat m1 m2 # 同时显示文件ml和m2的内容）
cat m1 m2 > file  #（将文件ml和m2合并后放入文件file中）
```