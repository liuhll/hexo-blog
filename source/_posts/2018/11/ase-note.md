---
title: 系统的性能，可用性，可伸缩性，可扩展性的概念
date: 2018-11-29 21:22:46
categories: "架构设计"
tags:
- 架构设计
---

# 性能(Performance)
性能是一个网站能够同时处理用户请求的表现能力。 不同的视觉，有不同的表现形式，性能的指标通常包括，*响应时间*，*并发数*，*吞吐量*，以及*性能计数器*等。

## 响应时间

系统响应时间，是计算机对用户的输入或请求作出反应的时间 。系统响应时间的计算要考虑到用户的数目，用户数目越多，响应时间必须越快，不然就难以保证每一个用户都有可以接受的响应时间。响应时间和时间片的大小有关，一般情况是：时间片越短，响应时间越快。

## 并发数
是指同时访问服务器站点的连接数。

## 吞吐量
指的就是单位时间内，系统处理的请求数量。 TPS（每秒的事务数），HPS（每秒的HTTP请求数），QPS（每秒的查询数）等等。性能一般通过缓存来解决。

## 性能计数器
性能计数器，它描述的是服务器或者操作系统的一组指标，包括，对象与线程数，内存使用，CPU使用，磁盘和网络的I/O等等。

## 提高性能的手段
提高网站的性能，很多的手段，比如，浏览器访问优化，CDN加速，反向代理，分布式缓存，使用集群，代码和数据结构的优化，存储性能的优化等

# 可用性(Availability)
可用性是在某个考察时间，系统能够正常运行的概率或时间占有率期望值。是衡量设备在投入使用后实际使用的效能，是设备或系统的可靠性、可维护性和维护支持性的综合特性。在大型网站应用系统中，衡量的指标一般是服务的可用性用几个9来表示。

## 瞬时可用性
考察时间为指定瞬间。

## 时段可用性
考察时间为指定时段。

## 固有可用性
考察时间为连续使用期间的任一时刻。

## 实现手段
高可用性一般通过负载均衡，数据备份，失效转移，提高软件质量，特别是发布时的质量来实现和保证的。


# 可伸缩性（Scalability）
是一种对软件系统计算处理能力的设计指标，高可伸缩性代表一种弹性，在系统扩展成长过程中，软件能够保证旺盛的生命力，通过很少的改动甚至只是硬件设备的添置，就能实现整个系统处理能力的线性增长，实现高吞吐量和低延迟高性能。


## 纵向的可伸缩性
在同一个逻辑单元内增加资源来提高处理能力。这样的例子包括在现有服务器上增加CPU，或者在现有的RAID/SAN存储中增加硬盘来提高存储量。

## 横向的可伸缩性
增加更多逻辑单元的资源，并令它们像是一个单元一样工作。大多数集群方案、分布式文件系统、负载平衡都是在帮助你提高横向的可伸缩性。

## 实现手段
可伸缩性一般通过DNS域名解析负载均衡，反向代理负载均衡，IP负载均衡，数据链路层负载均衡，改进和提高分布式缓存的算法，利用NOSQL数据库的可伸缩性等等

# 可扩展性（Extensibility）
可扩展性，通常和可伸缩性混为一谈.在软件范畴上，是软件系统本身的属性，或者进一步说是设计的属性，代码的属性。因为我们经常说设计的可扩展性，代码的可扩展性.也可以说是系统设计的松耦合性。

## 实现手段
实现方式：一般通过事件驱动架构和分布式架构来实现一个网站系统的可扩展性。


# 参考资料
- [并发连接数、请求数、并发用户数](https://blog.csdn.net/zhangdaiscott/article/details/60578450)
