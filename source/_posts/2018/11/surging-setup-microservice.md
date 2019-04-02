---
title: 微服务框架Surging之创建微服务项目
date: 2018-11-05 20:46:10
categories: "微服务"
tags:
  - 微服务
  - Surging
  - 开源框架
---


# 建立微服务项目
一般地，我们通过DDD领域设计来构建一个微服务组件。在开发过程中，我们可以单独为某个微服务组件建立解决方案，针对某个微服务进行独立的开发;但是更多的场景是将多个互相依赖的微服务建立为同一个解决方案，然后通过`docker-compose`进行编排微服务，方便业务开发和调试。

当然，除此之外，也可以通过普通的三层或是CQRS模式来构建微服务,这里不展开讨论。

## 建立微服务项目(组件)
根据领域设计架构模型，我们一般将微服务组件分为如下四层:
1. **Host Layer**: 主机服务层, 该层主要是用于寄宿(托管)微服务主机,在该层构建、配置和微服务微服务主机本身。
2. **IApplication Layer**：应用接口层， 该层主要定义微服务组件的**服务**和**命令**(概念类似于MVC中的`Controller`和`Action`的概念),并通过特性来标识服务与命令的相关属性(更多知识点，请参考[配置](#))。除此之外,每个服务对应的`DTO`对象，也在该层定义。
3. **Application Layer**: 应用层, 很薄的一层，`IApplication`的实现层,只包含工作流控制逻辑，不包含业务逻辑。
4. **Domain Layer**: 领域层,包含整个应用的所有业务逻辑。

当然，除了上述的四层之外，一般领域设计还可能包含: 

5. **Infrastructure Layer**: 基础设施层, 提供整个应用的基础服务；
6. **ORM Layer**: 数据访问层, 提供统一的仓储接口和对数据库的访问；

一般地，在一个产品(解决方案)中，我们会将基础设施层和数据访问层抽象出来，每个微服务组件只需要关注微服务自身的业务。除此之外，一个产品(解决方案)中,还会抽象出: 

7. **公共的业务组件**(相当于公共的领域层): 用于封装产品的通用业务。
8. **公共的Model组件**: 用于封装通用的值对象、枚举等。

一个通用的产品的解决方案结构应当如下图所示,其中,Services目录下为该解决方案(产品)的所有微服务组件:

![项目结构](sln-structure.png)

如果一个微服务组件作为一个单一的项目在开发，只需要定义 1~4 层即可,我们通过 Nuget 服务来安装使用到的组件包,换句话说，每个微服务只需要关注微服务自身的业务，而不需要关注除了本身之外的知识点,每个微服务组件的开发都应当满足：**高内聚、低耦合**的编程理念。

一般情况下，在开发过程中，往往一个大的产品可能会被分解为多个模块进行开发，相应的，一个模块会对应着一个解决方案，在这个解决方案内部定义了该模块的微服务组件，往往这些微服务组件具有较强的依赖关系，配合着完成该模块的业务，在这种场景下，微服务之间的通信建议采用通过接口代理的方式进行通信;而不同的模块之间的微服务之间通信，往往采用RPC的方式进行通信。更多内容,请参考[微服务通信方式](#)一节。

单个微服务组件的项目结构:

![单个微服务的项目结构](single.micoservice.structure.png)

> Notes:
> - 在开发过程中,我们应当把用于websocket通知的服务组件单独抽象出来。

## 构建微服务主机
在建立完项目后，下一步并开始着手构建微服务主机。Surging使用`ServiceHostBuilder`构建微服务主机, 在微服务主机构建完成后，执行 `Run()`方法。下面通过实例来描述如何构建微服务主机:

### 安装Surging组件包
在安装Surging组件包之前，首先需要配置内网的Nuget服务,内网的Nuget仓储地址为: `http://192.168.0.202/nuget`。
1. 一般情况下，以下组件包是必须安装的：`Surging.Core.CPlatform`、`Surging.Core.Common`、`Surging.Core.DotNetty`、`Surging.Core.Caching`;

2. 如果选择Consul作为服务注册中心,需要安装: `Surging.Core.Consul`组件包; 如果选择Zookeeper作为服务注册中心,则需要安装: `Surging.Core.Zookeeper`组件包;

3. 如果选择NLog作为日志记录组件，需要安装: `Surging.Core.NLog`组件包,如果选择log4net作为日志记录组件,则需要安装: `Surging.Core.Log4net`组件;

4. 如果微服务组件需要对外提供Http服务，则需要安装`Surging.Core.KestrelHttpServer`组件包。一般情况下，如果微服务组件提供Http服务，则需要通过Swagger生成在线API文档，需要安装`Surging.Core.Swagger`组件包。

5. 如果微服务主机作为 `WebSocket`服务主机，则需要安装`Surging.Core.Protocol.WS`组件包。

6. 编码器的选择: Surging默认使用Json格式作为消息序列化格式,性能无法达到最优,所以Surging提供了`Surging.Core.Codec.MessagePack`和`Surging.Core.Codec.ProtoBuffer`组件包作为消息序列化的编解码器,以提高性能，可以根据实际情况选择合适的消息编解码器。

### 编写配置文件
Surging架构采用Json格式编写配置文件，配置文件可以分为如下几类：
1. Surging平台配置文件:`surgingSettings.json`, 主要针对微服务主机自身的Ip、端口号、通信采用的协议、加载的模块，以及Swagger文档，日志输出级别进行配置。
2. 服务注册中心配置文件: `consul.json`或`zookeeper.json`，主要针对微服务的注册中心进行配置。
3. 消息中间件配置文件: `eventBusSettings.json`，针对微服务内部之间采用事件总线的中间件进行配置。
4. 缓存中间件配置文件: `cacheSettings.json`，针对微服务采用的缓存中间件进行配置。
5. 日志配置文件: `NLog.config`或`log4net.config`，日志配置文件按NLog 或 log4net组件的要求进行配置即可。

配置文件的模板可以通过[Surging的案例](https://gitee.com/GFCM/Surging.Core/tree/develop/src/Sample/Services/DigitalReleaseEval/Surging.Demo.DigitalReleaseEvalHost/Configs)进行拷贝,针对微服务配置的相关知识，请参考[配置](#)一节。

### 微服务主机类型
微服务主机的构建过程基本差不多，只是因为加载的模块不同(通过`surgingSettings.json`配置文档中的`Packages`节点)具有提供不同服务的能力，我们可以将微服务分为如下几种类型:
1. 业务微服务主机： 该类型的微服务主机不对外提供服务，只针对微服务内部集群提供服务,只要微服务主机注册到服务中心后,其他微服务主机可以通过接口代理或是RPC远程调用该主机的提供的接口。

2. 对外提供Http服务的微服务主机: 对客户端提供Http访问的能力,与第一类主机基本一致,只需要针对配置`surgingSettings.json`配置`Ports.HttpPort`端口号,并且在`Packages`节点下的`EnginePartModule`增加对`KestrelHttpModule`引用,如果需要生产Swagger文档,则需要安装`Surging.Core.Swagger`组件,并在`surgingSettings.json`中对`Swagger`节点进行配置。

3. WebSocket服务的微服务主机: 针对客户端提供websocket服务,该类型的主机在构建过程与第一类主机一致,但是必须要安装`Surging.Core.Protocol.WS`组件包，并且需要针对配置`surgingSettings.json`配置`Ports.WSPort`端口号，并且在`Packages`节点下的`EnginePartModule`增加对`WSProtocolModule`引用,需要注意的是：
    - 实现服务接口时,应用层需要继承的基类是`WSServiceBase`;
    - 与ws服务之间的调用只能通过RPC的方式调用，不支持通过接口创建代理远程调用;

4. 定时任务调度的微服务主机: 暂未实现

构建微服务主机的代码如下:

```csharp
var host = new ServiceHostBuilder()
                .RegisterServices(builder =>
                { // 注册服务
                    builder.AddMicroService(option =>
                    { // 增加微服务
                        option.AddServiceRuntime() // 增加服务运行时必要的组件
                        .AddRelateServiceRuntime() // 增加关联服务运行时必要的组件,如健康检查组件、地址选择器、远程调用服务组件等等
                        .AddClientProxy() // 如果需要支持通过接口代理进行其他微服务,则必须添加                        
                        .AddServiceEngine(typeof(SurgingServiceEngine));// 指定服务引擎,用于解析微服务的服务与命令

                        builder.Register(p => new CPlatformContainer(ServiceLocator.Current));
                    });
                })
                .ConfigureLogging(loggging =>
                {
                    loggging.AddConfiguration(
                        AppConfig.GetSection("Logging"));
                })
               // .UseNLog("NLog.config") // 指定日志中间件，可以不显示指明,直接在surgingSettings中进行配置日志组件即可
                .UseServer(options => { })
                .UseConsoleLifetime() // 指定控制台输出日志
                .Configure(build =>
                {
                    // 指定配置文件
                    build.AddCacheFile("${cachepath}|Configs/cacheSettings.json", optional: false, reloadOnChange: true);
                    build.AddCPlatformFile("${surgingpath}|Configs/surgingSettings.json", optional: false, reloadOnChange: true);
                    build.AddEventBusFile("Configs/eventBusSettings.json", optional: false);
                    build.AddConsulFile("Configs/consul.json", optional: false, reloadOnChange: true);

                })
                .UseProxy() // 声明可以使用接口代理方式访问服务
                .UseStartup<Startup>() //指定启动类
                .Build(); //构建微服务主机

            using (host.Run())
            {
                Console.WriteLine($"服务端启动成功，{DateTime.Now}。");
            }

```

1. 在构建过程中，需要指定`SurgingServiceEngine`用于解析服务组件,Surging采用**Sidecar模式**设计,如果未显示的引用应用组件,微服务可以通过服务引擎在指定的目录下扫描相应的dll，从而解析到相应的服务与命令。
2. 必须指定`Startup`类。
3. 需要注意的是注意`Configs`目录的大小写,Windows系统对目录的大小写不敏感,而Linux系统对大小写是敏感的。

## 编写Dockerfile
如果需要使用docker-compose或k8s来编排微服务，则必须要构建微服务的镜像,可以通过如下脚本来构建微服务组件的Docker镜像。

```dockerfile
FROM microsoft/dotnet:2.1-runtime AS base
WORKDIR /app
ARG rpc_port=100
ARG http_port=8080
ARG ws_port=96
EXPOSE ${rpc_port} ${http_port} ${ws_port}

FROM microsoft/dotnet:2.1-sdk AS build
WORKDIR /src
COPY . .
WORKDIR <micro_service_project_path>
RUN dotnet restore && \
    dotnet build --no-restore -c Release -o /app

FROM build AS publish
RUN dotnet publish --no-restore -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "<micro_service_name.dll>"]
```
