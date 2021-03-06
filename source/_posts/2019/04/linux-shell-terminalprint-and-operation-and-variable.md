---
title: 终端打印、算术运算、常用变量
date: 2019-04-09 16:54:56
categories: "linux"
tags:
- shell
---

# 终端打印

## echo
```shell
echo hello world
echo 'hello world' # bash不会对单引号内变量（如$var）求值。
echo "hello world"
```
## printf
- 更为强大打印命令支持格式替换符
```shell
#!/bin/bash

printf "%-5s %-10s %-4s\n" NO Name Mark
printf "%-5s %-10s %-4.2f\n" 01 Tom 90.3456
printf "%-5s %-10s %-4.2f\n" 02 Jack 89.2345
printf "%-5s %-10s %-4.2f\n" 03 Jeff 98.4323
```

### 格式替代符
- `%b` 相对应的参数被视为含有要被处理的转义序列之字符串。
- `%c` ASCII字符。显示相对应参数的第一个字符
- `%d`, %i 十进制整数
- `%e`, %E, %f 浮点格式
- `%g` %e或%f转换，看哪一个较短，则删除结尾的零
- `%G` %E或%f转换，看哪一个较短，则删除结尾的零
- `%o` 不带正负号的八进制值
- `%s` 字符串
- `%u` 不带正负号的十进制值
- `%x` 不带正负号的十六进制值，使用a至f表示10至15
- `%X` 不带正负号的十六进制值，使用A至F表示10至15
- `%%` 字面意义的%

### 转义序列
- `\a` 警告字符，通常为ASCII的BEL字符
- `\b` 后退
- `\c` 抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，- `都被`忽略
- `\f` 换页（formfeed）
- `\n` 换行
- `\r` 回车（Carriage return）
- `\t` 水平制表符
- `\v` 垂直制表符
- `\\` 一个字面上的反斜杠字符
- `\ddd` 表示1到3位数八进制值的字符，仅在格式字符串中有效
- `\0ddd` 表示1到3位的八进制值字符

# 算术运算

## 整数运算
### 1. let运算命令
```shell
#!/bin/bash
no1=2;
no2=3;
let result=no1+no2
echo $result
```
- 自加操作`let no++`
- 自减操作`let no--`
- 简写形式`let no+=10` `let no-=20`分别等同于`let no=no+10` `let no=no-20`

### 2. 操作符[]运算方法
```shell
#!/bin/bash
no1=2;
no2=3;
result=$[$no1+no2]
echo $result
```
- 使用方法和let相似，在[]中可以使$前缀。

### 3. (())运算方法
```shell
#!/bin/bash
no1=2;
no2=3;
result=$((no1+no2))
echo $result
```
### 4. expr运算方法
```shell
result=`expr 2 + 3`
result=$(expr $no1 + 5)
```
**expr的常用运算符**
- 加法运算：`+`
- 减法运算：`-`
- 乘法运算：`\*`
- 除法运算：`/`
- 求摸（取余）运算：`%`

## 精密计算
算术操作高级运算工具：`bc`，它可以执行浮点运算和一些高级函数

例如:

```shell
echo "1.212*3" | bc 
3.636
```
### 设定小数精度（数值范围）
```shell
echo "scale=2;3/8" | bc # 参数scale=2是将bc输出结果的小数位设置为2位
0.37
```

### 进制转换
```shell
#!/bin/bash
abc=192
echo "obase=2;$abc" | bc
# 执行结果为：11000000，这是用bc将十进制转换成二进制。

#!/bin/bash
abc=11000000
echo "obase=10;ibase=2;$abc" | bc
# 执行结果为：192，这是用bc将二进制转换为十进制。
```

### 计算平方和平方根
```shell
echo "10^10" | bc
echo "sqrt(100)" | bc
```

# 常用变量
结合不同的引导为变量赋值
- 双引号 `""` ：允许通过$符号引用其他变量值
- 单引号 `''` ：禁止引用其他变量值，`$`视为普通字符
- 反撇号 `` ：将命令执行的结果输出给变量

## 用户自定义变量

### 设置变量的作用范围

格式：
```shell
export 变量名...
export 变量名=变量值 [...变量名n=变量值n]
```
### 清除用户自定义变量

格式：
```shell
unset 变量名
```

## 环境变量
### 环境变量配置文件
- 全局配置文件：`/etc/profile`
- 用户配置文件：`~/.bash_profile`

### 查看环境变量
`set`命令可以查看所有的shell变量，其中包括环境变量

### 常见的环境变量
- $USER 查看账户信息
- $logname 登录相关信息
- $UID
- $Shell
- $HOME 家目录
- $pwd
- $PATH 用户所输入的命令是在哪些目录中查找
- $PS1
- $PS2
- $RANDOM 随机数

## 位置变量
表示为：`$n` （n为1~9之间的数字）
```shell
#./test.sh one two three four five six
```
- `$0` 表示文件名本身
- `one`就是：`$1`
- `two`就是：`$2`

## 预定义变量
- `$#`：命令行中位置参数的个数
- `$*`：所有位置参数的内容
- `$?`：上一条命令执行后返回的状态，当返回状态值为0时表示执行正常，非0表示执行异常或出错
- `$$`：当前所在进程的进程号
- `$!`：后台运行的最后一个进程号
- `$0`：当前执行的进程/程序名