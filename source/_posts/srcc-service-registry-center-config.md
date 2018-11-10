---
title: 微服务框架Srcc之服务注册中心配置
date: 2018-11-10 19:46:01
categories: "微服务"
tags:
  - 微服务
  - surging
  - Srcc
  - 开源框架
---

SRCC 支持Consul和Zookeeper作为服务注册中心。

# Consul配置

| 配置项 | 说明 | 备注 |
|:-------------------|:-----------------|:---------------|
| ConnectionString | Consul服务注册中心地址 | |
| SessionTimeout | 会话超时时间 | |
| RoutePath | 服务组件地址 | 服务引擎会通过指定该地址扫描到相应的服务组件，并解析到相应的服务与命令 |
| ReloadOnChange | 当配置改变后是否重新加载 | |

# Zookeeper配置
略