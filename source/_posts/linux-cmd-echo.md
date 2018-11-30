---
title: echo命令
date: 2018-11-30 21:36:08
categories: "linux"
tags:
- linux命令
---

# 简介
`echo`命令用于在shell中打印shell变量的值，或者直接输出指定的字符串。linux的echo命令，在shell编程中极为常用, 在终端下打印变量value的时候也是常常用到的，因此有必要了解下echo的用法echo命令的功能是在显示器上显示一段文字，*一般起到一个提示的作用*。

# 语法
```shell
echo(选项)(参数)
```

# 选项
```
-e：激活转义字符
```

使用`-e`选项时，若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出：
- \a 发出警告声；
- \b 删除前一个字符；
- \c 最后不加上换行符号；
- \f 换行但光标仍旧停留在原来的位置；
- \n 换行且光标移至行首；
- \r 光标移至行首，但不换行；
- \t 插入tab；
- \v 与\f相同；
- \\ 插入\字符；
- \nnn 插入nnn（八进制）所代表的ASCII字符；

# 参数
变量：指定要打印的变量。

# 实例

1. 文字色

```shell
echo -e "\e[1;31mThis is red text\e[0m"
This is red text # shell终端会输出红色的字体

```
- `\e[1;31m` 将颜色设置为红色
- `\e[0m` 将颜色重新置回

2. 背景色

```shell
echo -e "\e[1;42mGreed Background\e[0m"
Greed Background # 字体背景为绿色
```