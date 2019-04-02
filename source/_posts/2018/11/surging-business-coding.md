---
title: 微服务框架Surging之编写业务代码
date: 2018-11-05 21:01:13
categories: "微服务"
tags:
  - 微服务
  - Surging
---


本节主要描述了如何编写业务代码，当前，每个微服务组件主要通过领域设计分层设计,下面的每个小节都给出了每一层的目录结构，建议开发过程中严格按照给定的目录结构进行编写业务代码。特别是对领域设计不是很熟悉的情况下，对快速上手开发是很有帮助的。领域设计秉承了**高内聚、低耦合**的原则。特别不建议在不熟悉领域设计的情况下，对目录结构进行创新。否则，后期项目将会变得非常混乱，且难以维护。

## 应用层接口层
在定义服务接口之前，我们需要安装`Surging.Core.CPlatform`组件包。

### 定义服务

一般情况下,编写某一个业务模块的Api时,我们需要在应用接口层定义一个接口,该接口继承自`IServiceKey`，并通过`ServiceBundle`特性来标识该服务的路由模板(类似MVC中的特性路由)。
例如:

```csharp
    [ServiceBundle("v1/demo/{service}")]
    public interface IDemoApplication : IServiceKey
    {
        
    }
```

### 定义命令
在定义服务之后，就需要在服务内部定义命令，每一个命令就是一个接口方法。服务与命令的关系就类似与MVC框架中的`Controller`和`Action`。

所有命令都应当是异步的。可以通过`CommandAttribute`对命令进行注解,从而影响命令在运行时的行为，如果不对命令进行注解,那么运行时解析命令将会以缺省值进行标记。

命令的声明如下所示:

```csharp
[Command(Strategy = StrategyType.Injection, ShuntStrategy = AddressSelectorMode.HashAlgorithm, ExecutionTimeoutInMilliseconds = 2500, BreakerRequestVolumeThreshold = 3, Injection = @"return 1;", RequestCacheEnabled = false)]
Task<string> SayHello(string name);
```

`CommandAttribute`配置相关参数列表:

| 参数 | 作用 | 备注 | 
|:------------------|:--------------------------|:----------------|
| FailoverCluster | 故障转移次数 |  默认为3次  |
| CircuitBreakerForceOpen | 是否强制开启熔断 | 默认为`false`  |
| Strategy | 容错策略 | 系统提供三种容错策略: <br> 1.Failover, 失败切换远程服务机制 <br> 2. Injection,失败执行注入脚本(通过`Injection`属性注入) <br> 3. FallBack,失败执行指定回调方法，指定的方法必须要集成自`IFallbackInvoker`接口 |
| ExecutionTimeoutInMilliseconds | 执行超时时间	 | 默认值：1000 |
| RequestCacheEnabled |	是否开启缓存 |	默认关闭 |
| Injection | 脚本注入 |	 |
| InjectionNamespaces | 注入命名空间 | 称为程序集名称更恰当 |
| BreakeErrorThresholdPercentage | 错误率达到多少开启熔断保护 |	默认值：50 |
| BreakeSleepWindowInMilliseconds |	熔断多少秒后去尝试请求 |	默认值：60000 |
| BreakerForceClosed |	是否强制关闭熔断	 | |
| BreakerRequestVolumeThreshold | 10秒钟内至少多少请求失败，熔断器才发挥起作用 |	默认值：20 |
| MaxConcurrentRequests | 最大并发数 |	10 |

### DTO对象
DTO对象英文名称为: Data Transfer Object,中文名为:数据传输对象。那么，[为什么要使用DTO对象呢?](https://www.cnblogs.com/Gyoung/archive/2013/03/23/2977233.html),简而言之，DTO对象不同于领域层的中实体(Entity)和值对象(Model),领域层中的实体对象和值对象是面向业务的，而DTO对象是面向UI的。
另外，我们将DTO对象定义在应用接口层是因为，如果微服务组件之间有强的依赖关系，这个时候我们可以通过接口代理的方式访问其他微服务组件，将DTO对象定义在应用接口层而不是应用层是为了方便引用DTO对象。

一般而言,DTO对象可以加动词前缀和形容词后缀,例如: `GetXxxxOutput`、`QueryXxxxInput`、`XxxxDto`等均为合法的DTO命名规范。

### 应用接口层的项目结构

一个理想的应用层接口的目录结构应当如下所示:

```
|-- ModuleName
|-- |-- IModuleApplication.cs
|-- |-- Dto
|------ |-- GetModuleOutput.cs
|------ |-- CreateModuleInput.cs
|------ |-- ModuleDto.cs
```

## 应用层
应用层是对应用接口层定义的接口进行实现的，是一层很薄的一层，**只应当包含工作流控制逻辑，不包含业务逻辑**。

一般地，微服务之间的通信只应当发生在该层,如果微服务之间具有强的依赖关系，例如A服务组件强依赖B服务，那么A服务可以引用B服务的应用接口层，并且可以通过接口代理的方式访问B服务组件的接口。如果服务与服务之间并没有依赖关系的话，那么服务之间的通信就应当采用RPC的方式进行通信。关于微服务之间的通信请参考[通信一节](../communication-mode/index.md)

如果需要使用接口代理的方式访问其他微服务组件接口，那么在应用层需要安装`Surging.Core.ProxyGenerator`组件包，并且需要在构建微服务主机的时候指明使用代理,请参考[构建微服务主机](./setup-microservice.md#微服务主机类型)一节；如果只需要通过RPC代理的方式访问其他微服务组件接口，那么在应用层只需要安装`Surging.Core.CPlatform`组件包。

一般的，应用类需要继承`ProxyServiceBase`基类。如果应用是提供的基于`WebSocket`服务的话,那么应用层需要安装`Surging.Core.Protocol.WS`组件包,并且应用类需要继承的是`WSServiceBase`基类,且对应的命令的负载均衡算法必须显示的标注为`哈希算法`,且第一个参数必须为`string`类型(原因: Ws服务通过第一个参数进行hash的)。
例如:

```csharp
        /// <summary>
        /// 数字化放行评估过程中的消息
        /// </summary>
        /// <returns></returns>
        [Command(ShuntStrategy = AddressSelectorMode.HashAlgorithm)]
        Task<bool> EvalMessage(string key,EvalMessage evalMessage);
```

除此之外,ws服务之间的调用只能通过基于routepath远程(RPC调用)调用，不支持通过接口创建代理远程调用。

**特别要注意的是:**千万不要在应用层加入业务逻辑，应用层应当只包括工作流程控制逻辑，如果涉及业务逻辑，那么应当分析是否是该微服务领域内的业务，如果是，业务实现逻辑应当在该微服务组件的领域层，如果不是，应当调用其他微服务组件的接口，通过接口访问执行相关业务或获取相关业务数据。换句话说，应用层关注的仅仅是:第一步做什么，第二步做什么,而不关心应该怎么做;该怎么做，如何做，应当是领域层关心的事。**如果是做简单的查询，不包含任何的业务逻辑，请直接通过实体或聚合根的仓储类查询数据。**

除此之外，应用层往往还需要一个单独的静态类用于定义引用到其他微服务组件的接口名称。

一般情况下，应用层的目录结构如下所示:
```
|-- ModuleName
|---- |-- ModuleApplication.cs
|-- WebApiConfig.cs 
```

## 领域层
领域层是具体的业务实现的一层，关注的是业务功能的实现。领域层应当根据业务模块进行建模,更多领域知识，请参考[汤雪华博客:DDD理论积累](https://www.cnblogs.com/netfocus/category/361987.html)


领域层需要安装`Surging.Core.CPlatform`组件包。领域层需要注意的是，需要定义一个模块类,将定义的对象注入到相关的`Ioc容器`中，并且在`surgingsetting.json`中显示配置引用该模块。
例如:
模块类定义为:
```csharp
    public class OperationRestrictionDomainModule : BusinessModule
    {
        protected override void RegisterBuilder(ContainerBuilderWrapper builder)
        {
            // 该部分待重构,真实项目中通过标识接口实现自动注册
            builder.RegisterType<FlightInfoRepository>()
                .As<IRepository<FlightInfoEntity>>()
                .AsSelf()
                .InstancePerDependency();
            builder.RegisterType<FlightInfoManager>()
                .As<IFlightInfoManager>()
                .InstancePerDependency();
            builder.RegisterType<OperationRestricEvalManager>()
                .As<IOperationRestricEvalManager>()
                .InstancePerDependency();
        }
    }

```
主机服务中需要设置:
```json
    "Packages": [
      {
        "TypeName": "EnginePartModule",
        "Using": "${UseEngineParts}|DotNettyModule;NLogModule;ConsulModule;WSProtocolModule;EventBusRabbitMQModule;CachingModule;KestrelHttpModule"
      },
      {
        "TypeName": "BusinessModule",
        "Using": "OperationRestrictionDomainModule;DemoCoreModule;"
      }
    ]
```

> **Notes**
> - 该部分后期会优化, 领域层的组件注册设计为通过继承标识接口`ITransientDependency`和`ISingletonDependency`来自动注入到Ioc容器中。

一般地，领域层的目录结构可能如下所示(领域对象采用贫血模式):
```
|-- ModuleName
|---- |-- IModuleManager.cs
|---- |-- ModuleManager.cs
|---- |-- XxxPolicy.cs
|---- |-- OtherBusinessClass.cs // 代表其他业务类代码,例如：某个模块采用了命令模式,该部分可以看做是Commands的集合
|---- |-- Entities
|---- |----- |---- Xxx1Aggregate.cs  // 每一个模块仅有只有仅有一个聚合根，但可以有多个实体对象，多个值对象
|---- |----- |---- Xxx1Repository.cs
|---- |----- |---- Xxx2Entity.cs
|---- |----- |---- Xxx2Repository.cs
|---- |----- |---- Xxx3Entity.cs
|---- |----- |---- Xxx3Repository.cs
|---- |-- ValueObjects
|---- |----- |---- XxxModel.cs
|---- |----- |---- XxxValueObject.cs
|---- |----- |---- XxxEnum.cs
|-- ModuleNameModule.cs 
```

> **Notes**
> - 一般情况下，如果封装的好，在`ModuleManager`对象中直接引用仓储的泛型接口,除非对仓储进行扩展，否则并不需要定义仓储类。
> - `ModuleManager`是该领域内的具体的业务实现的执行者。
> - 一般的，Entities目录下定义聚合根，实体对象，ValueObjects目录下定义值对象、枚举值等

## 公共的领域层
一个解决方案(业务模块)，有可能需要抽象出一个公共的领域层，用于处理该业务模块的公共业务。例如: 事件处理、消息代理、服务代理等等。
除此之外，我们还会定义一个公共的基础实施层、和公共的model层。

## 运行项目
在运行项目之前，必须保证运行环境已经提供了服务注册中心(Consul或Zookeeper)，消息中间件(RabbitMq或Kafka)，以及缓存中间件。一般地，如果不同团队之间需要联调微服务，应当在局域网内部部署相应的服务或中间件；而在独立开发的场景下，建议采用docker-compose编排微服务。




