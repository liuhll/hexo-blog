---
title: Ocelot学习笔记
date: 2018-09-24 21:56:31
categories: "微服务"
tags: 
  - 网关
  - 微服务
---

## 简介
Ocelot是一个用.NET Core实现并且开源的API网关，它功能强大，包括了：**路由**、**请求聚合**、**服务发现**、**认证**、**鉴权**、**限流熔断**、并内置了 **负载均衡器** 与 **Service Fabric** 、**Butterfly Tracing** 集成。

本质上,Ocelot是一堆的asp.net core [middleware]( https://github.com/TomPallister/Ocelot/blob/develop/src/Ocelot/Middleware/OcelotMiddlewareExtensions.cs) 组成的一个管道。当它拿到请求之后会用一个`request builder`来构造一个`HttpRequestMessage`发到下游的真实服务器，等下游的服务返回`response`之后再由一个`middleware`将它返回的`HttpResponseMessage`映射到`HttpResponse`上。

- asp.net core中间件示意图

![moddleware](https://blog.johnwu.cc/images/pasted-114.gif)

### API网关
**API网关**—— 它是系统的暴露在外部的一个访问入口。这个有点像代理访问，就像一个公司的门卫承担着寻址、限制进入、安全检查、位置引导、等等功能。

![API网关路径](http://www.jessetalk.cn/wp-content/uploads/2018/03/APISM-768x349.png)

## 路由
配置文件中的`ReRoutes`可以实现由上游到下游路由的转发。

### 上游路由
API网关匹配到的请求路由地址。

### 下游路由
API网关转发到真是服务主机的真实服务地址。

### 路由负载均衡
当下游服务有多个结点的时候，我们可以在`DownstreamHostAndPorts`中进行配置:

```json
{
    "DownstreamPathTemplate": "/api/posts/{postId}",
    "DownstreamScheme": "https",
    "DownstreamHostAndPorts": [
            {
                "Host": "10.0.1.10",
                "Port": 5000,
            },
            {
                "Host": "10.0.1.11",
                "Port": 5000,
            }
        ],
    "UpstreamPathTemplate": "/posts/{postId}",
    "LoadBalancer": "LeastConnection",
    "UpstreamHttpMethod": [ "Put", "Delete" ]
}
```



## 部署

### 基本使用
![基本使用](https://ocelot.readthedocs.io/en/latest/_images/OcelotBasic.jpg)
### 与 `IdentityServer` 集成
当我们涉及到认证和鉴权的时候，我们可以跟`Identity Server`进行结合。当网关需要请求认证信息的时候会与`Identity Server`服务器进行交互来完成

![与indentityserver集成](https://ocelot.readthedocs.io/en/latest/_images/OcelotIndentityServer.jpg)
### 网关集群
只有一个网关是很危险的，也就是我们通常所讲的单点，只要它挂了，所有的服务全挂。这显然无法达到高可用，所以我们也可以部署多台网关。
![集群](https://ocelot.readthedocs.io/en/latest/_images/OcelotMultipleInstances.jpg)
### Consul服务发现
在Ocelot已经支持简单的负载功能，也就是当下游服务存在多个结点的时候，Ocelot能够承担起负载均衡的作用。但是它不提供健康检查，服务的注册也只能通过手动在配置文件里面添加完成。这不够灵活并且在一定程度下会有风险。这个时候我们就可以用Consul来做服务发现，它能与Ocelot完美结合。

![Consul服务发现](https://ocelot.readthedocs.io/en/latest/_images/OcelotMultipleInstancesConsul.jpg)

## 配置
Ocelot有两个配置块。一个`ReRoutes`数组和一个`GlobalConfiguration`。
- `ReRoutes`配置块是一些告诉Ocelot如何处理上游请求的对象
- `Globalconfiguration`有些奇特，可以覆盖`ReRoute`节点的特殊设置。如果你不想管理大量的`ReRoute`特定的设置的话，这将很有用。

```shell
{
    "ReRoutes": [],
    "GlobalConfiguration": {}
}
```
`ReRoutes`路由配置例子

```shell
{
          "DownstreamPathTemplate": "/",
          "UpstreamPathTemplate": "/",
          "UpstreamHttpMethod": [
              "Get"
          ],
          "AddHeadersToRequest": {},
          "AddClaimsToRequest": {},
          "RouteClaimsRequirement": {},
          "AddQueriesToRequest": {},
          "RequestIdKey": "",
          "FileCacheOptions": {
              "TtlSeconds": 0,
              "Region": ""
          },
          "ReRouteIsCaseSensitive": false,
          "ServiceName": "",
          "DownstreamScheme": "http",
          "DownstreamHostAndPorts": [
              {
                  "Host": "localhost",
                  "Port": 51876,
              }
          ],
          "QoSOptions": {
              "ExceptionsAllowedBeforeBreaking": 0,
              "DurationOfBreak": 0,
              "TimeoutValue": 0
          },
          "LoadBalancer": "",
          "RateLimitOptions": {
              "ClientWhitelist": [],
              "EnableRateLimiting": false,
              "Period": "",
              "PeriodTimespan": 0,
              "Limit": 0
          },
          "AuthenticationOptions": {
              "AuthenticationProviderKey": "",
              "AllowedScopes": []
          },
          "HttpHandlerOptions": {
              "AllowAutoRedirect": true,
              "UseCookieContainer": true,
              "UseTracing": true
          },
          "UseServiceDiscovery": false,
          "DangerousAcceptAnyServerCertificateValidator": false
      }

```


## 请求聚合

### 什么是聚合路由?
把多个正常的ReRoutes打包并映射到一个对象来对客户端的请求进行响应。

比如，你请求订单信息，订单中又包含商品信息，这里就设计到两个微服务，一个是商品服务，一个是订单服务。如果不运用聚合路由的话，对于一个订单信息，客户端可能需要请求两次服务端。实际上这会造成服务端额外的开销。这时候有了聚合路由后，你只需要请求一次聚合路由，然后聚合路由会合并订单跟商品的结果都一个对象中，并把这个对象响应给客户端。使用Ocelot的此特性可以让你很容易的实现前后端分离的架构。

为了实现Ocelot的请求功能，你需要在`ocelot.json`中进行如下的配置。这里我们指定了了两个正常的`ReRoutes`,然后给每个`ReRoute`设置一个`Key`属性。最后我们再`Aggregates`节点中的`ReRouteKeys`属性中加入我们刚刚指定的两个`Key`从而组成了两个`ReRoutes`的聚合。当然我们还需要设置`UpstreamPathTemplate`匹配上游的用户请求，它的工作方式与正常的`ReRoute`类似。
> **Notes:** 不要把Aggregates中UpstreamPathTemplate设置的跟ReRoutes中的UpstreamPathTemplate设置成一样

例如:
1. 存在两个微服务: `OrderApi` 和 `GoodApi`

```csharp
//GoodApi项目中
    [Route("api/[controller]")]
    [ApiController]
    public class GoodController : ControllerBase
    {
        // GET api/Good/5
        [HttpGet("{id}")]
        public ActionResult<string> Get(int id)
        {
            var item = new Goods
            {
                Id = id,
                Content = $"{id}的关联的商品明细",
            };
            return JsonConvert.SerializeObject(item);
        }
    }
  //OrderApi项目中  
    [Route("api/[controller]")]
    [ApiController]
    public class OrderController : ControllerBase
    {
        // GET api/Order/5
        [HttpGet("{id}")]
        public ActionResult<string> Get(int id)
        {
            var item = new Orders {
                Id=id,
                Content=$"{id}的订单明细",
            };
            return JsonConvert.SerializeObject(item);
        }
    }
```

2. 分别在`ocelot.good.json`以及`ocelot.order.json`中新增一个路由，并给出`Keys`
```csharp
//ocelot.good.json
{
      "DownstreamPathTemplate": "/api/Good/{id}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 1001
        }
      ],
      "UpstreamPathTemplate": "/good/{id}",
      "UpstreamHttpMethod": [ "Get", "Post" ],
      "Key": "Good",
      "Priority": 2
    }
//ocelot.order.json
{
      "DownstreamPathTemplate": "/api/Order/{id}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 1002
        }
      ],
      "UpstreamPathTemplate": "/order/{id}",
      "UpstreamHttpMethod": [ "Get", "Post" ],
      "Key": "Order",
      "Priority": 2
}
```

3. 在`ocelot.all.json`中加入聚合配置

```csharp
"Aggregates": [
    {
      "ReRouteKeys": [
        "Good",
        "Order"
      ],
      "UpstreamPathTemplate": "/GetOrderDetail/{id}"
    }
  ]
```

> **Notes:** 这里`Aggregates`跟`ReRoutes`同级，`ReRouteKeys`中填写的数组就是上面步骤3中设置的`Key`属性对应的值。

4. 客户端请求接口地址 `http://localhost:1000/GetOrderDetail/1` 叫得到如下响应

```csharp
{
    "Good":{
        "Id":1,
        "Content":"1的关联的商品明细"
    },
    "Order":{
        "Id":1,
        "Content":"1的订单明细"
    }
}
```

## 服务发现

Ocelot允许您指定服务发现提供程序，并将使用它来查找Ocelot将请求转发到的下游服务的主机和端口。目前，这仅在`GlobalConfiguration`部分中受支持，这意味着相同的服务发现提供程序将用于为`ReRoute`级别指定`ServiceName`的所有`ReRoutes`。

### 使用步骤

1. 安装Consul支持包

```shell
Install-Package Ocelot.Provider.Consul
```

在`startup`类中的`ConfigureServices`方法中增加

```csharp
services.AddOcelot()//注入Ocelot服务
                    .AddConsul(); 
```

2. `GlobalConfiguration`中需要加入以下内容。如果您未指定主机和端口，则将使用`Consul`默认值。

```json
"ServiceDiscoveryProvider": {
    "Host": "localhost",
    "Port": 8500,
    "Type": "Consul"
}
```


