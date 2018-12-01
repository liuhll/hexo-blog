---
title: Shell脚本简介
date: 2018-12-01 22:05:41
categories: "linux"
tags:
- shell脚本
---

# 什么是shell
在计算机系统中,能够控制计算机硬件（CPU、内存、显示器等）的只有操作系统内核（Kernel），无论是图形界面还是命令行都只是架设在用户和内核之间的一座桥梁。

由于安全、复杂、繁琐等原因，用户不能直接接触内核（也没有必要），需要另外再开发一个程序，让用户直接使用这个程序；该程序的作用就是接收用户的操作（点击图标、输入命令），并进行简单的处理，然后再传递给内核。如此一来，用户和内核之间就多了一层“代理”，这层“代理”既简化了用户的操作，也保护了内核。用户界面和命令行就是这个另外开发的程序，就是这层“代理”。在Linux下，这个命令行程序叫做 Shell。

# shell的作用
- 调用其他程序，给其他程序传递数据或参数，并获取程序的处理结果；
- 在多个程序之间传递数据，把一个程序的输出作为另一个程序的输入；
- Shell 本身也可以被其他程序调用。

总的来说: Shell 是将内核、程序和用户连接了起来。

# shell脚本的解释器
Linux 的 Shell 种类众多，常见的有：
- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）

> **Notes**
> `Bash` 是大多数Linux 系统默认的 Shell。

在一般情况下，人们并不区分 Bourne Shell 和 Bourne Again Shell，所以，像 `#!/bin/sh`，它同样也可以改为 `#!/bin/bash`
`#!` 告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序

# 运行shell脚本的方法
## 作为可执行程序
1. 使脚本具有执行权限
2. 执行脚本

```shell
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

> **Notes**
> 一定要写成 `./test.sh`，而不是 `test.sh`，运行其它二进制的程序也一样，直接写 `test.sh`，linux 系统会去 `PATH` 里寻找有没有叫 `test.sh` 的，而只有 `/bin`, `/sbin`, `/usr/bin`，`/usr/sbin` 等在 `PATH` 里，你的当前目录通常不在 `PATH` 里，所以写成 `test.sh` 是会找不到命令的，要用 `./test.sh` 告诉系统说，就在当前目录找。

## 作为解释器参数
- 直接运行解释器，其参数就是 shell 脚本的文件名

```shell
/bin/sh test.sh
/bin/php test.php
```
> **Notes**
> 这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用