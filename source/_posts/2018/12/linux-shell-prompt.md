---
title: Shell提示符($和#的区别)
date: 2018-12-02 21:55:20
categories: "linux"
tags:
- shell脚本
---

# 简介
启动终端模拟包或者从 Linux 控制台登录后，便可以看到 Shell 提示符。提示符是通往 Shell 的大门，是输入 Shell 命令的地方。

## 普通用户的提示符`$`
普通用户的提示符是美元符号`$`。

## 超级用户提示符`#`
超级用户(`root`)的提示服务好井号`#`。

# 提示符格式
不同的 Linux 发行版使用的提示符格式不同。例如在 CentOS 中，默认的提示符格式为：
```shell
[liunx@liuhl-ubuntu ~]$
```

ubuntu的提示符格式为:
```shell
liuhl@liuhl-ubuntu:~$
```
提示符格式一般包含三方面的信息:
- 启动 Shell 的用户名，也即 liuhl；
- 本地主机名称，也即 liuhl-ubuntu
- 当前目录，波浪号`~`是主目录的简写表示法。

Shell 通过`PS1`和`PS2`两个环境变量来控制提示符格式:
- `PS1` 控制最外层命令行的提示符格式。
   ```shell
   liuhl@liuhl-ubuntu:~$ echo $PS1
   \[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\u@\h:\w\$
   ```
- `PS2` 控制第二层命令行的提示符格式。
   ```shell
   liuhl@liuhl-ubuntu:~$ echo $PS2
   >
   liuhl@liuhl-ubuntu ~$ echo "
   > yan
   > chang
   > sheng
   > "
   ```

Shell 使用以`\`为前导的特殊字符来表示命令提示符中包含的要素，这使得 `PS1` 和 `PS2` 的格式看起来可能有点奇怪。下表展示了可以在 `PS1` 和 `PS2` 中使用的特殊字符。

| 字符 | 描述 |
|:-----|:-------|
| \a | 铃声字符 |
| \d | 格式为“日 月 年”的日期 |
| \e | ASCII转义字符 |
| \h | 本地主机名 |
| \H | 完全合格的限定域主机名 |
| \j | shell当前管理的作业数 |
| \1 | shell终端设备名的基本名称 |
| \n | ASCII换行字符 |
| \r | ASCII回车 |
| \s | shell的名称 |
| \t | 格式为“小时:分钟:秒”的24小时制的当前时间 |
| \T | 格式为“小时:分钟:秒”的12小时制的当前时间 |
| \@ | 格式为am/pm的12小时制的当前时间 |
| \u | 当前用户的用户名 |
| \v | bash shell的版本 |
| \V | bash shell的发布级别 |
| \w | 当前工作目录 |
| \W | 当前工作目录的基本名称 |
| \! | 该命令的bash shell历史数 |
| \# | 该命令的命令数量 |
| \$ | 如果是普通用户，则为美元符号$；如果超级用户（root 用户），则为井号#。 |
| \nnn | 对应于八进制值 nnn 的字符 |
| \\ | 斜杠 |
| \[ | 控制码序列的开头 |
| \] | 控制码序列的结尾 |

## 修改提示符
```shell
[liuhl@localhost ~]$ PS1="[\t][\u]\$ "
[22:14:34][liuhl]$ 
```
新的 Shell 提示符现在可以显示当前的时间和用户名。
不过这个新定义的 `PS1` 变量只在当前 Shell 会话期间有效，再次启动 Shell 时将重新使用默认的提示符格式。