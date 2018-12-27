---
title: Shell数组：shell数组的定义、数组长度
date: 2018-12-27 22:34:27
categories: "linux"
tags:
- shell脚本
---
# 简介
Shell在编程方面比Windows批处理强大很多，无论是在循环、运算。

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。类似与C语言，数组元素的下标由`0`开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于`0`。

# 定义数组

在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。定义数组的一般形式为：
    `array_name=(value1 ... valuen)`
例如：

```shell
array_name1=(value0 value1 value2 value3)
# 或者
array_name2=(
value0
value1
value2
value3
)
# 还可以单独定义数组的各个分量：

array_name3[0]=value0
array_name3[1]=value1
array_name3[2]=value2
```
可以不使用连续的下标，而且下标的范围没有限制。

# 读取数组
读取数组元素值的一般格式是：
`${array_name[index]}`

```shell
valuen=${array_name[2]}
```
## 例子
```shell
#!/bin/sh
NAME[0]="Zara"
NAME[1]="Qadir"
NAME[2]="Mahnaz"
NAME[3]="Ayan"
NAME[4]="Daisy"
echo "First Index: ${NAME[0]}"
echo "Second Index: ${NAME[1]}"
# 输出
# First Index: Zara
# Second Index: Qadir
```

## 使用`@` 或 `*` 可以获取数组中的所有元素
```shell
${array_name[*]}
${array_name[@]}
```
- **例子**
```shell
#!/bin/sh
NAME[0]="Zara"
NAME[1]="Qadir"
NAME[2]="Mahnaz"
NAME[3]="Ayan"
NAME[4]="Daisy"
echo "First Method: ${NAME[*]}"
echo "Second Method: ${NAME[@]}"
# 输出
# First Method: Zara Qadir Mahnaz Ayan Daisy
# Second Method: Zara Qadir Mahnaz Ayan Daisy
```
# 获取数组的长度
获取数组长度的方法与获取字符串长度的方法相同。
```shell
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```