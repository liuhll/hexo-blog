---
title: 微服务框架Surging之简介
date: 2018-11-3 21:07:20
categories: "微服务"
tags:
  - 微服务
  - Surging
  - 开源框架
---


# 架构简介
Surging是基于.net core2.1平台,在开源社区github[Surging](https://github.com/dotnetcore/surging)的一个开源框架。

该框架提供高性能的RPC远程服务调用,服务引擎支持http、TCP、WS协议,采用Zookeeper或Consul作为服务的注册中心，集成了哈希、随机、轮询、压力最小优先作为负载均衡的算法，RPC集成采用的是高性能的netty通信框架，数据通信采用异步传输,具有**高可用**、**高性能**、**可扩展**的特性。每个微服务组件均可通过水平扩展来应对高并发。

Surging的简单案例[Surging.Sample](https://github.com/liuhll/Surging.Sample)可以参考我的github。

## 快速开始
1. {% post_link surging-development-env 搭建开发环境 %}
2. {% post_link surging-setup-microservice 创建微服务项目 %}  
3. {% post_link surging-orchestration-microservice 使用`docker-compose`编排微服务 %}
4. {% post_link surging-business-coding 编写业务代码 %} 

## Surging框架的通信方式
1. {% post_link surging-request-response 请求-异步响应 %}
2. {% post_link surging-publish-subscribe 发布-异步订阅 %} 
3. {% post_link surging-notice 通知 %} 

## Surging框架配置
1. {% post_link surging-config 微服务组件配置 %}
2. {% post_link surging-service-registry-center-config 服务注册中心配置 %}
3. {% post_link surging-message-middleware-config 消息中间件配置 %}
4. {% post_link surging-cache-middleware-config 缓存中心件配置 %} 
