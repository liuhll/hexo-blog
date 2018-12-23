---
title: 微服务框架Srcc之消息中间件配置
date: 2018-11-10 19:50:51
categories: "微服务"
tags:
  - 微服务
  - surging
  - Srcc
  - 开源框架
---

SRCC框架支持RabbitMq和Kafka中间件。

# RabbitMq中间件配置
| 配置项 | 说明 | 备注 |
|:-------------------|:-----------------|:---------------|
| EventBusConnection | RabbitMq服务地址|  |
| EventBusUserName | 用户名 |  |
| EventBusPassword | 密码 |  |
| VirtualHost | 虚拟主机 |  |
| MessageTTL | 消息过期时间，比如过期时间是30分钟就是1800000 |  |
| RetryCount | 重试次数 |  |
| FailCount | 处理失败流程重试次数，如果出现异常，会进行重试 |  |
| PrefetchCount | 设置均匀分配消费者消息的个数 |  |
| BrokerName | BrokerName |  |
| Port | RabbitMq服务端口号 |  |

RabbitMq参考资料:
- https://www.jianshu.com/p/cd15278fa38b
- http://rabbitmq.mr-ping.com/

# Kafka中间件配置

| 配置项 | 说明 | 备注 |
|:-------------------|:-----------------|:---------------|
| Servers | |  |
| MaxQueueBuffering | |  |
| MaxSocketBlocking | |  |
| EnableAutoCommit | |  |
| LogConnectionClose | |  |
| OffsetReset | |  |
| GroupID | |  |

Kafka参考资料:
- http://orchome.com/kafka/index