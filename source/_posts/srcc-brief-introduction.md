---
title: 微服务框架Srcc之简介
date: 2018-11-3 21:07:20
categories: "微服务"
tags:
  - 微服务
  - surging
  - Srcc
  - 开源框架
---


# 架构简介
SRCC是基于.net core2.1平台,在开源框架[Surging](https://github.com/dotnetcore/surging)的基础上进行二次开发的微服务框架。

该框架提供高性能的RPC远程服务调用,服务引擎支持http、TCP、WS协议,采用Zookeeper或Consul作为服务的注册中心，集成了哈希、随机、轮询、压力最小优先作为负载均衡的算法，RPC集成采用的是高性能的netty通信框架，数据通信采用异步传输,具有**高可用**、**高性能**、**可扩展**的特性。每个微服务组件均可通过水平扩展来应对高并发。

SRCC 英文全称为: `Service Run Control Center`,中文名称为: `服务运行控制中心` 。 该框架是我为当前所在公司研发的产品所选用的技术框架，并在原有的Surging的框架做了些微调，且新增了分布式任务调度组件包、AutoMapper组件包、数据校验组件包、以及对领域层、数据访问层进行了抽象。

## 快速开始
1. {% post_link srcc-development-env 搭建开发环境 %}
2. {% post_link srcc-setup-microservice 创建微服务项目 %}  
3. {% post_link srcc-orchestration-microservice 使用`docker-compose`编排微服务 %}
4. [编写业务代码](#)

## SRCC框架的通信方式
1. [请求-异步响应](#)
2. [发布-异步订阅](#)
3. [通知](#)

## SRCC框架配置
1. [微服务组件配置](#)
2. [服务注册中心配置](#)
3. [消息中间件配置](#)
4. [缓存中心件配置](#)
5. [日志组件配置](#)