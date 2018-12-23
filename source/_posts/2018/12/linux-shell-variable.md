---
title: Shell变量
date: 2018-12-11 21:20:56
categories: "linux"
tags:
- shell脚本
---

# Shell变量简介
在`Bash shell` 中，每一个变量的值都是**字符串**，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。这意味着，`Bash shell` 在默认情况下不会区分变量类型，即使你将整数和小数赋值给变量，它们也会被视为字符串，这一点和大部分的编程语言不同。
脚本语言在定义变量时通常不需要指明类型，直接赋值就可以，Shell 变量也遵循这个规则。

# 定义变量
Shell 支持以下三种定义变量的方式：
```shell
variable=value
variable='value'
variable="value"
```
- `variable` 是变量名，`value` 是赋给变量的值
- 如果 `value` 不包含任何空白符（例如空格、Tab缩进等），那么可以不使用引号；
- 如果 `value` 包含了空白符，那么就必须使用引号包围起来


> **Notes**
> - 赋值号的周围不能有空格，这与绝大多数的编程语言不同。

## 命名规范
- 变量名由数字、字母、下划线组成；
- 必须以字母或者下划线开头；
- 不能使用 Shell 里的关键字（通过 `help` 命令可以查看保留关键字）。

##  单引号和双引号的区别
- 以单引号`' '`包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出。这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景。
- 以双引号`" "`包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量名和命令原样输出。这种方式比较适合字符串中附带有变量和命令并且想将其解析后再输出的变量定义。

**例子**

```shell
#!/bin/bash
url="https://liuhll.github.io/hexo-blog-deploy/"
website1='我的个人博客网站：${url}'
website2="我的个人博客网站:${url}"
echo $website1
echo $website2
# 输出
# 我的个人博客网站：${url}
# 我的个人博客网站:https//liuhll.github.io/hexo-blog-deploy/
```

## 使用建议
- 如果变量的内容是数字，那么可以不加引号；
- 如果真的需要原样输出就加单引号；
- 其他没有特别要求的字符串等最好都加上双引号，定义变量时加双引号是最常见的使用场景。

# 使用变量
使用一个定义过的变量，只要在变量名前面加美元符号`$`即可。
```shell
author="刘洪亮"
echo $author
echo ${author}
```

- 变量名外面的花括号`{ }`是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界。
```shell
skill="java"
echo "I am good at ${skill}Script"
```
如果不给`skill` 变量加花括号，写成`echo "I am good at $skillScript"`，解释器就会把 `$skillScript `当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

> **Notes**
> 推荐给所有变量加上花括号`{ }`，这是个良好的编程习惯

# 修改变量的值
- 已定义的变量，可以被重新赋值。
- 第二次对变量赋值时不能在变量名前加`$`，只有在使用变量时才能加`$`。

```shell
url="https://liuhll.github.io"
echo ${url}
url="https://liuhll.github.io/hexo-blog-deploy/"
echo ${url}
```

# 将命令的结果赋值给变量
Shell 也支持将命令的执行结果赋值给变量，常见的有以下两种方式
```shell
variable=`command`
variable=$(command)
```
1. 第一种方式把命令用反引号包围起来，反引号和单引号非常相似，容易产生混淆，所以不推荐使用这种方式；
2. 第二种方式把命令用`$()`包围起来，区分更加明显，所以推荐使用这种方式。

# 只读变量
使用 `readonly` 命令可以将变量定义为只读变量，*只读变量的值不能被改变*。
- 尝试更改只读变量，结果报错。

```shell
#!/bin/bash
myUrl="https://liuhll.github.io"
readonly myUrl
myUrl="https://liuhll.github.io/hexo-blog-deploy/"
# error output => /bin/sh: NAME: This variable is read only.
```

# 删除变量
- 使用 `unset` 命令可以删除变量，变量被删除后不能再次使用
- unset 命令不能删除只读变量。

```shell
#!/bin/sh
myUrl="https://liuhll.github.io/hexo-blog-deploy/"
unset myUrl
echo $myUrl
# 脚本没有任何输出
```

# 变量类型

## 1.局部变量
局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。

## 2.环境变量
所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。

## 3.shell变量
shell变量是由shell程序设置的*特殊变量*。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行
