---
title: sosreport命令
date: 2018-12-27 22:14:03
categories: "linux"
tags:
- linux命令
---

# 简介
`sosreport`是一个类型于`supportconfig` 的工具，`sosreport`是python编写的一个工具，适用于centos（和redhat一样，包名为sos）、ubuntu（其下包名为sosreport）等大多数版本的linux 。`sosreport`在github上的托管页面为：https://github.com/sosreport/sos ，而且默认在很多系统的源里都已经集成有。如果使用的是正版redhat，在出现系统问题，寻求官方支持时，官方一般也会通过sosreport将收集的信息进行分析查看。需要注意的是在一些老的redhat发行版中叫`sysreport` ------ 如redhat4.5之前的版本中。


# sosreport的安装
## centos / redhat
```shell
yum -y insatll sos
```

##  ubuntu下的安装
```shell
sudo apt-get install sosreport
```
# sosreport用法
可以使用`sosreport --help`或`man sosreport` 获取使用帮助手册

## sosreport命令进行系统环境收集
```shell
sosreport -a --report
```