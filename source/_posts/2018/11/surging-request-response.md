---
title: 微服务框架Surging之请求响应通信
date: 2018-11-06 19:35:14
categories: "微服务"
tags:
  - 微服务
  - Surging
  - 开源框架
---


微服务集群内部最主要的通信方式，应对微服务之间的绝大多数通信场景。该通信模式主要是采用基于netty的（IPC）进程通信，是基于请求/异步响应的IPC机制。Netty是一个非常优秀的通信框架,其用法和架构设计可参考[Netty 实战(精髓)](https://waylau.com/essential-netty-in-action/) 。

服务之间的调用方式主要有两种方式：
1. 通过接口代理方式
2. 基于routepath的RPC远程调用

# 接口代理方式
## 适用场景
微服务之间有强的依赖关系。例如：微服务组件A强依赖微服务B,那么，微服务组件A是可以通过引用微服务组件B的应用接口层，通过服务代理工厂`IServiceProxyFactory`创建微服务组件B的业务接口的代理，通过代理对象直接调用微服务组件B的业务方法。

## 优缺点
### 优点
1. 用法简单、直接；通过接口代理访问与一般的调用方法一致。

### 缺点
1. 微服务组件有着强耦合的关系，不满足高内聚、低耦合的编程原则。如果微服务之间有强的依赖关系，建议采用该种通信方式，否则建议使用RPC录用调用的方式进行通信。
2. WS服务不支持通过接口创建代理远程调用

## 用法
1. 在A服务组件中引用B服务组件的应用接口层。
2. 在构建微服务主机时显示声明使用代理方法。
```csharp
 var host = new ServiceHostBuilder()
                .RegisterServices(builder =>
                { 
                    builder.AddMicroService(option =>
                    { 
                        option // 其他方法略
                        .AddClientProxy()
                        builder.Register(p => new CPlatformContainer(ServiceLocator.Current));
                    });
                })
                .UseProxy() // 其他方法略
                .Build();
```
3. 通过服务代理工厂创建B服务组件的业务接口
```csharp
//  调用天气保障服务评估当前天气是否可放行
    var weatherEvalProxy = GetService<IServiceProxyFactory>()
        .CreateProxy<IWeatherEvalApplication>();
    var weatherResult = await weatherEvalProxy.WeatherEval(new FlightInfoInput()
    {
        FlightDate = flightInfo.FlightDate,
        FlightId = flightInfo.FlightId
    });
```

# 基于routepath的RPC远程调用

## 适用场景
微服务组件A与微服务组件B之间的关系是并列的，例如：两个不同业务模块之间的微服务组件之间的通信，建议采用该种方式进行通信。

## 优缺点
### 优点
1. 服务与服务之间没有引用关系，不产生耦合性。
2. 构造微服务主机时无需声明使用代理类。

### 缺点
1. 需要自己构建参数，用法较麻烦。
2. 无法指定返回的数据类型，默认返回的是object类型，建议使用`dynamic`作为返回结果。

## 用法
1. 使用字典类构建请求参数`Dictionary<string, object>`,请求参数的名称必须与应用接口的名称一致。
2. 通过服务代理提供者`IServiceProxyProvider`调用接口，传入请求参数和接口的路由地址。

```csharp
var requestParams = new Dictionary<string, object>() {
    { "input", 
       new { flightDate = flightInfo.FlightDate, flightId = flightInfo.FlightId } 
    } 
    };
var weatherResult =  await GetService<IServiceProxyProvider>().Invoke<bool>(requestParams, "v1/weathereval//weathereval");
```

> **Notes**
> - 实际编码过程中，应当避免硬编码,每个服务组件调用的接口地址都应当维护到一个静态类中,[请参考](../quick-start/business-coding.md#应用层)。