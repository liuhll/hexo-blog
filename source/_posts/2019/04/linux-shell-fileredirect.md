---
title: 文件的描述符和重定向
date: 2019-04-10 15:17:30
categories: "linux"
tags:
- shell
---

# 简介
文件描述符是和文件的输入、输出相关联的非负整数，Linux内核（kernel）利用文件描述符（file descriptor）来访问文件。打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件。常见的文件描述符是`stdin`、`stdout`和`stderr`。

# 系统预留文件描述符
- 0 —— stdin（标准输入）
- 1 —— stdout（标准输出）
- 2 —— stderr（标准错误）

## 重定向将输入文本通过截取模式保存到文件：
```shell
echo "this is a text line one" > test.txt
# 写入到文件之前，文件内容首先会被清空。
```
## 重定向将输入文本通过追加模式保存到文件：
```shell
echo "this is a text line one" >> test.txt
# 写入到文件之后，会追加到文件结尾。
```
## 标准错误输出：
```shell
[root@localhost text]# cat linuxde.net
cat: linuxde.net: No such file or directory
```

## 标准错误输出的重定向方法：
```shell
# 方法一：
[root@localhost text]# cat linuxde.net 2> out.txt  //没有任何错误提示，正常运行。
# 方法二：
[root@localhost text]# cat linuxde.net &> out.txt

# 因为错误信息被保存到了out.txt文件中。
```
将错误输出丢弃到`/dev/null`中，`/dev/null`是一个特殊的设备文件，这个文件接受到任何数据都会被丢系，通常被称为位桶、黑洞。

```shell
[root@localhost text]# cat linuxde.net 2> /dev/null
```

# tee命令
`tee`命令可以将数据重定向到文件，另一方面还可以提供一份重定向数据的副本作为后续命令的`stdin`。

![tee](tee.png)

## 例子
在终端打印stdout同时重定向到文件中：
```shell
ls | tee out.txt
1.sh
1.txt
2.txt
eee.tst
EEE.tst
one
out.txt
string2
www.pdf
WWW.pdf
WWW.pef
# 显示行号
[root@localhost text]# ls | tee out.txt | cat -n
 1  1.sh
 2  1.txt
 3  2.txt
 4  eee.tst
 5  EEE.tst
 6  one
 7  out.txt
 8  string2
 9  www.pdf
10  WWW.pdf
11  WWW.pef
```

# 重定向脚本内的文本片段（多行文本）
```shell
#!/bin/bash
cat <<EOF>text.log
this is a text line1
this is a text line2
this is a text line3
EOF
```
在`cat <<EOF>text.log`与下一个`EOF`行之间的所有文本都会当作`stdin`数据输入到text.log中。

# 自定义文件描述符
除了`0`、`1`和`2`分别是`stdin`、`stdout`和`stderr`的系统预留描述符，我们还可以使用`exec`命令创建自定义文件描述符，文件的的打开模式有只读模式、截断模式和追加模式。

## < 操作符用于从文件中读取至stdin：
```shell
echo this is a test line > input.txt
exec 3<input.txt    //自定义文件描述符3打开并读取文件
# 在命令中使用文件描述符3：

cat <&3
this is a test line
# 这里需要注意只能读取一次，如果再次使用需要重新创建文件描述符。
```

## > 操作符用于截断模式的文件写入（数据在文件内容被截断之后写入）：
```shell
exec 4>output.txt
echo this is a new line >&4
cat output.txt
this is a new line
```
## >> 操作符用于追加模式的文件写入（添加数据到文件中，原有数据不会丢失）：
```shell
exec 5>>output.txt
echo this is a appended line >&5
cat output.txt
this is a new line
this is a appended lin
```