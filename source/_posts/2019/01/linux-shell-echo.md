---
title: Shell echo命令
date: 2019-01-06 17:09:20
categories: "linux"
tags:
- shell脚本
---

# 简介
`echo`是Shell的一个内部指令，用于在屏幕上打印出指定的字符串。命令格式：
```shell
echo arg
```
# 用法案例

## 转义字符 `\`
```shell
echo "\"It is a test\""
```
## 显示变量 `$`
```shell
name="OK"
echo "$name It is a test"
```

## 换行
```shell
echo "OK!\n"
echo "It is a test"
# 输出：
# OK!
# It si a test
```
# 不换号
```shell
echo "OK!\c"
echo "It is a test"
#输出：
#OK!It si a test
```

# 重定向文件
```shell
echo "It is a test" > myfile
```
# 原样输出
- 若需要原样输出字符串（不进行转义），请使用单引号
```shell
echo '$name\"'
```

# 命令执行结果
```shell
echo `date`
# 或是
echo $(date)
```
# 参考
- [echo命令](https://liuhll.github.io/hexo-blog-deploy/2018/11/30/2018-12-linux-cmd-echo/)