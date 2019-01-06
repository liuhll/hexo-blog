---
title: pwd命令
date: 2019-01-06 16:57:29
categories: "linux"
tags:
- linux命令
---
# 简介
`pwd`命令以绝对路径的方式显示用户当前工作目录。命令将当前目录的全路径名称（从`根目录`）写入标准输出。全部目录使用`/`分隔。第一个`/`表示根目录，最后一个目录是当前目录。执行`pwd`命令可立刻得知您目前所在的工作目录的绝对路径名称。

# 语法
```shell
pwd（选项）
```

# 选项
```shell
--help：显示帮助信息；
--version：显示版本信息。
```

# 实例
```shell
[root@localhost ~]# pwd
/root
```