---
title:  微服务框架Srcc之缓存中间件配置
date: 2018-11-10 19:56:00
categories: "微服务"
tags:
  - 微服务
  - surging
  - Srcc
---

通过指定`CachingSettings.Class`来选择配置的缓存中间件的上下文参数,当前Srcc支持两种类型的缓存中间件:
1. Redis
2. MemoryCache


缓存中间件的配置项如下所述:

| 配置项 | 说明 | 备注 |
|:-------------------|:-----------------|:---------------|
| CachingSettings.Id | 缓存中间件的唯一标识 | |
| CachingSettings.Class | 缓存中间件的的上下文参数,格式为: `缓存中间件上下文,程序集名称` | |
| CachingSettings.InitMethod | 指定初始化的方法 | |
| CachingSettings.Maps |  | |
| CachingSettings.Properties | 配置缓存中间件上下文的参数 | 参数类型为数组 |

# redis配置
CachingSettings.Class 配置项的值为: `SunWin.SRCC.Caching.RedisCache.RedisContext,SunWin.SRCC.Caching`
除此之外，通过CachingSettings.Properties 配置redis的链接、账号、密码、默认到期时间、连接超时等属性。

# MemoryCache
内存缓存，默认的缓存中间件，不支持分布式应用。

# 缓存中间件的扩展
可以通过扩展`CachingSettings.Class` 和 `ICacheProvider` 来扩展指定类型的缓存中间件。