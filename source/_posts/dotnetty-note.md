---
title: Dotnetty笔记
date: 2018-10-09 19:16:57
categories: "网络框架"
tags:
- 开源框架
- 网络通信
---

## DotNetty介绍
DotNetty是微软的Azure团队，使用C#实现的`Netty`的版本发布。不但使用了C#和.Net平台的技术特点，并且保留了`Netty`原来绝大部分的编程接口。让我们在使用时，完全可以依照`Netty`官方的教程来学习和使用DotNetty应用程序。

dotNetty 是一个非阻塞、事件驱动的网络框架。dotNetty 实际上是使用 Threads（多线程）处理 I/O 事件,`DotNetty`是一个`NIO`客户端服务端框架，它使得快速而简单的开发像服务端客户端协议的网络应用成为了可能。它极大的简化并流线化了如`TCP`和`UDP`套接字服务器开发的网络编程。`dotNetty`成功找到了一个无需折衷妥协而让开发、性能、稳定性和灵活性相互协调的方法。

## 类库简介
1. DotNetty.Buffers     
  - 对内存缓冲区管理的封装。
2. DotNetty.Codecs   
  - 对编解码是封装，包括一些基础基类的实现，在项目中自定义的协议，都要继承该项目的特定基类和实现。
3. DotNetty.Codecs.Mqtt
  - `MQTT`（消息队列遥测传输）编解码是封装，包括一些基础基类的实现。
4. DotNetty.Codecs.Protobuf
  - `Protobuf`编解码是封装，包括一些基础基类的实现。
5. DotNetty.Codecs.ProtocolBuffers
  - `ProtocolBuffers`编解码是封装，包括一些基础基类的实现。
6. DotNetty.Codecs.Redis
  - `Redis`协议编解码是封装，包括一些基础基类的实现。
7. DotNetty.Common
  - 是公共的类库项目，包装线程池，并行任务和常用帮助类的封装。
8. DotNetty.Handlers
  - 封装了常用的管道处理器，比如`Tls`编解码，超时机制，心跳检查，日志等。
9. DotNetty.Transport
  - 是`DotNetty`核心的实现，`Socket`基础框架，通信模式：异步非阻塞。
10. DotNetty.Transport.Libuv
  - 是`DotNetty`自己实现基于`Libuv` （高性能的，事件驱动的I/O库） 核心的实现。

## 架构图
![架构图](https://7n.w3cschool.cn/attachments/image/wk/netty4userguide/4.1.png)

## DotNetty核心概念
### BOOTSTRAP
`DotNetty` 应用程序通过设置 `Bootstrap`（引导）类的开始，该类提供了一个用于应用程序网络层配置的容器。

### CHANNEL
底层网络传输API必须提供给应用 I/O操作的接口，如读，写，连接，绑定等等。对于我们来说，这是结构几乎总是会成为一个“socket”。
`DotNetty` 中的接口`Channel` 定义了与 socket 丰富交互的操作集：`bind`, `close`, `config`, `connect`, `isActive`, `isOpen`, `isWritable`, `read`, `write` 等等。 `DotNetty` 提供大量的 `Channel` 实现来专门使用。这些包括 `AbstractChannel`，`AbstractNioByteChannel`，`AbstractNioChannel`，`EmbeddedChannel`， `LocalServerChannel`，`NioSocketChannel` 等等。

### CHANNELHANDLER
`ChannelHandler` 支持很多协议，并且提供用于数据处理的容器。我们已经知道`ChannelHandler` 由特定事件触发。 `ChannelHandler` 可专用于几乎所有的动作，包括将一个对象转为字节（或相反），执行过程中抛出的异常处理。

常用的一个接口是 `ChannelInboundHandler`，这个类型接收到入站事件（包括接收到的数据）可以处理应用程序逻辑。当你需要提供响应时，你也可以从 `ChannelInboundHandler` 冲刷数据。一句话，业务逻辑经常存活于一个或者多个 `ChannelInboundHandler`。

### CHANNELPIPELINE
`ChannelPipeline` 提供了一个容器给 `ChannelHandler` 链并提供了一个API 用于管理沿着链入站和出站事件的流动。每个 `Channel` 都有自己的`ChannelPipeline`，当`Channel` 创建时自动创建的。 `ChannelHandler` 是如何安装在 `ChannelPipeline`？ 主要是实现了`ChannelHandler` 的抽象 `ChannelInitializer`。`ChannelInitializer`子类 通过 `ServerBootstrap` 进行注册。当它的方法 `InitChannel()` 被调用时，这个对象将安装自定义的 `ChannelHandler` 集到 `pipeline`。当这个操作完成时，`ChannelInitializer` 子类则 从 `ChannelPipeline` 自动删除自身。

### EVENTLOOP
`EventLoop` 用于处理 `Channel` 的 I/O 操作。一个单一的 `EventLoop`通常会处理多个 `Channel` 事件。一个 `EventLoopGroup` 可以含有多于一个的` EventLoop` 和 提供了一种迭代用于检索清单中的下一个。

### CHANNELFUTURE
`DotNetty` 所有的 I/O 操作都是异步。因为一个操作可能无法立即返回，我们需要有一种方法在以后确定它的结果。出于这个目的，dotNetty 提供了接口 `ChannelFuture`,它的 `AddListener` 方法注册了一个 `ChannelFutureListener` ，当操作完成时，可以被通知（不管成功与否）。

## 引导

### Bootstrap 类型
`DotNetty`的包括两种不同类型的引导。 两个引导实现自一个名为`AbstractBootstrap`的超类。
#### 服务端引导
“服务器”应用程序把一个“父”管道接受连接和创建“子”管道,`ServerBootstrap` 中的 `ChildHandler()`, `ChildAttr()` 和 `ChildOption()` 是常用的服务器应用的操作。具体来说,`ServerChannel`实现负责创建子 `Channel`,它代表接受连接。因此 引导 `ServerChannel` 的 `ServerBootstrap` ,提供这些方法来简化接收的 `Channel` 对 `ChannelConfig` 应用设置的任务。
下图展示了服务端是如何工作的
![服务引导](https://waylau.com/essential-netty-in-action/images/Figure%209.3%20ServerBootstrap.jpg)


```csharp
// 主工作线程组，设置为1个线程
    var bossGroup = new MultithreadEventLoopGroup(1);
    // 工作线程组，默认为内核数*2的线程数
    var workerGroup = new MultithreadEventLoopGroup();
    X509Certificate2 tlsCertificate = null;
    if (ServerSettings.IsSsl) //如果使用加密通道
            {
                tlsCertificate = new X509Certificate2(Path.Combine(ExampleHelper.ProcessDirectory, "dotnetty.com.pfx"), "password");
            }
            try
            {
                
                //声明一个服务端Bootstrap，每个Netty服务端程序，都由ServerBootstrap控制，
                //通过链式的方式组装需要的参数

                var bootstrap = new ServerBootstrap();
                bootstrap
                    .Group(bossGroup, workerGroup) // 设置主和工作线程组
                    .Channel<TcpServerSocketChannel>() // 设置通道模式为TcpSocket
                    .Option(ChannelOption.SoBacklog, 100) // 设置网络IO参数等，这里可以设置很多参数，当然你对网络调优和参数设置非常了解的话，你可以设置，或者就用默认参数吧
                    .Handler(new LoggingHandler("SRV-LSTN")) //在主线程组上设置一个打印日志的处理器
                    .ChildHandler(new ActionChannelInitializer<ISocketChannel>(channel =>
                    { //工作线程连接器 是设置了一个管道，服务端主线程所有接收到的信息都会通过这个管道一层层往下传输
//同时所有出栈的消息 也要这个管道的所有处理器进行一步步处理
                        IChannelPipeline pipeline = channel.Pipeline;
                        if (tlsCertificate != null) //Tls的加解密
                        {
                            pipeline.AddLast("tls", TlsHandler.Server(tlsCertificate));
                        }
                        //日志拦截器
                        pipeline.AddLast(new LoggingHandler("SRV-CONN"));
//出栈消息，通过这个handler 在消息顶部加上消息的长度
                        pipeline.AddLast("framing-enc", new LengthFieldPrepender(2));
//入栈消息通过该Handler,解析消息的包长信息，并将正确的消息体发送给下一个处理Handler，该类比较常用，后面单独说明
                        pipeline.AddLast("framing-dec", new LengthFieldBasedFrameDecoder(ushort.MaxValue, 0, 2, 0, 2));
//业务handler ，这里是实际处理Echo业务的Handler
                        pipeline.AddLast("echo", new EchoServerHandler());
                    }));
            
        // bootstrap绑定到指定端口的行为 就是服务端启动服务，同样的Serverbootstrap可以bind到多个端口
                IChannel boundChannel = await bootstrap.BindAsync(ServerSettings.Port);

//关闭服务
                await boundChannel.CloseAsync();
            }
            finally
            {
//释放工作组线程
                await Task.WhenAll(
                    bossGroup.ShutdownGracefullyAsync(TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(1)),
                    workerGroup.ShutdownGracefullyAsync(TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(1)));
            }
```

1. 创建要给新的 `ServerBootstrap` 来创建新的 `SocketChannel` 管道并绑定他们
2. 指定 `EventLoopGroup` 用于从注册的 `ServerChannel` 中获取`EventLoop` 和接收到的管道
3. 指定要使用的管道类
4. 设置子处理器用于处理接收的管道的 I/O 和数据
5. 通过配置引导来绑定管道

#### 客户端引导
“客户端”很可能只需要一个单一的、非“父”对所有网络交互的管道。
- 如何引导客户端？
`Bootstrap`类负责创建管道给客户或应用程序，利用无连接协议和在调用 `Bind()` 或 `Connect()` 之后。
下图展示了如何工作
![客户端引导](https://waylau.com/essential-netty-in-action/images/Figure%209.2%20Bootstrap%20process.jpg)
```csharp
var group = new MultithreadEventLoopGroup();

            X509Certificate2 cert = null;
            string targetHost = null;
            if (ClientSettings.IsSsl)
            {
                cert = new X509Certificate2(Path.Combine(ExampleHelper.ProcessDirectory, "dotnetty.com.pfx"), "password");
                targetHost = cert.GetNameInfo(X509NameType.DnsName, false);
            }
var bootstrap = new Bootstrap();
                bootstrap
                    .Group(group)
                    .Channel<TcpSocketChannel>()
                    .Option(ChannelOption.TcpNodelay, true)
                    .Handler(new ActionChannelInitializer<ISocketChannel>(channel =>
                    {
                        IChannelPipeline pipeline = channel.Pipeline;

                        if (cert != null)
                        {
                            pipeline.AddLast("tls", new TlsHandler(stream => new SslStream(stream, true, (sender, certificate, chain, errors) => true), new ClientTlsSettings(targetHost)));
                        }
                        pipeline.AddLast(new LoggingHandler());
                        pipeline.AddLast("framing-enc", new LengthFieldPrepender(2));
                        pipeline.AddLast("framing-dec", new LengthFieldBasedFrameDecoder(ushort.MaxValue, 0, 2, 0, 2));

                        pipeline.AddLast("echo", new EchoClientHandler());
                    }));

                IChannel clientChannel = await bootstrap.ConnectAsync(new IPEndPoint(ClientSettings.Host, ClientSettings.Port));

                await clientChannel.CloseAsync();
```

1. 创建一个新的 Bootstrap 来创建和连接到新的客户端管道
2. 指定 EventLoopGroup
3. 指定 Channel 实现来使用
4. 设置处理器给 Channel 的事件和数据
5. 连接到远端主机

## 参考
- [Netty 实战(精髓)](https://waylau.com/essential-netty-in-action/)
- [Netty 4.x 用户指南](https://waylau.gitbooks.io/netty-4-user-guide/content/)
- [DotNetty项目基本了解和介绍](https://www.cnblogs.com/syxlb/p/8484698.html)
- [Netty源码解析](https://blog.csdn.net/z69183787/article/category/6412734/1)