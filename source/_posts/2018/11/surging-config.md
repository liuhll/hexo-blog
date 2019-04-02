---
title: 微服务框架Surging之组件配置
date: 2018-11-08 20:56:56
categories: "微服务"
tags:
  - 微服务
  - Surging
  - 开源框架
---


微服务的配置项可以通过指定的环境变量改写。微服务的配置采用Json文件来编写，配置格式如下所示:
```
配置项:"${环境变量名称}|默认值"
```

# Surging节点配置
该节点主要配置微服务主机运行时的相关参数,包括主机IP、端口号、更新路由的心跳间隔、运行的最大并发量等参数。

如下表所述:

| 配置项 | 说明 | 备注 |
|:-------------------|:-----------------|:---------------|
| Surging.Address | 配置微服务主机的Ip地址或域名 |  |
| Surging.WatchInterval | 设置向服务注册中心更新路由数据的心跳值 | 缺省值为20s |
| Surging.Port | 微服务主机运行的端口号 |  |
| Surging.MappingIp | 映射的Ip地址 | 暂不需要配置 |
| Surging.MappingPort | 映射的端口号 | 暂不需要配置 |
| Surging.Token | 是否验证Token | 缺省值为True |
| Surging.Libuv | 是否开启Libuv | 缺省值为false |
| Surging.Protocol | 配置微服务组件支持的协议 | 缺省值为: None; <br/>可选的值: <br> 1.None <br> 2. Tcp <br>3.Http <br> 4.Ws   |
| Surging.RootPath | 指定服务引擎解析的业务模块路径 | |
| Surging.Ports.HttpPort | 指定微服务对外提供的Http端口号 | 微服务组件必须依赖`KestrelHttpModule`才会对外提供Http服务，该配置项才有效 |
| Surging.Ports.WsPort | 指定微服务对外提供的Ws端口号 | 微服务组件必须依赖`WSProtocolModule`才会对外提供WS服务，该配置项才有效 |

除了上述配置项之外，Surging配置节点还支持对服务组件的命令进行缺省配置。Surging框架在解析命令时,会有优先采用命令的特性值，然后是Surging节点下的配置值，如果都不存在，则会使用缺省值。
关于命令的配置参数请参考[定义命令](../quick-start/business-coding.md#定义命令)一节。

# Swagger配置
只有微服务组件对外提供了Http服务，才可能生成Swagger文档。微服务必须依赖`KestrelHttpModule`和安装了`Surging.Core.Swagger`组件包，并且配置了Swagger相关参数，才可能生成SwaggerApi文档。

Swagger配置参数如下表所述:

| 配置项 | 说明 | 备注 |
|:-------------------|:-----------------|:---------------|
| Swagger.Version | Api文档的版本号 |  |
| Swagger.Title | Api文档标题 | 建议填写微服务组件的名称  |
| Swagger.Contact.Name | 联系人 | 建议填写微服务组件的开发责任人  |
| Swagger.Contact.Uri | 微服务组件的项目地址 |   |
| Swagger.Contact.Email | 联系人的Email |   |


# 日志级别配置
该节点主要配置日志组件输出日志的级别。日志组件的输出级别除了受该节点本身的影响之外，还受日志组件本身的配置文件影响。
日志级别的配置与 Asp.Net Core 的配置一致，在此不做赘述。

