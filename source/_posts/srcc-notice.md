---
title: 微服务框架Srcc通信之通知
date: 2018-11-08 20:07:49
categories: "微服务"
tags:
  - 微服务
  - surging
  - Srcc
  - 开源框架
---

SRCC框架支持Netty与websocket进行通信。

# 应用场景
WebSocket用于服务端向客户端向客户端推送消息。

# 用法
## 构建ws服务主机
请参考{% post_link srcc-setup-microservice 创建微服务项目 %}  

## 定义服务与命令。
Ws的服务与命令的定义与应用业务接口的定义一致，但是需要注意的是，命令的负载均衡算法必须选择哈希算法,命令的第一个参数必须为`string`类型,否则无法通过分布式部署ws服务(,Ws服务是通过命令接口的第一个参数的Hash值进行寻址的)。

例如:
```csharp
    [ServiceBundle("v1/messagenotify/{Service}")]
    public interface IMessageNotify : IServiceKey
    {
        /// <summary>
        /// 数字化放行评估过程中的消息
        /// </summary>
        /// <returns></returns>
        [Command(ShuntStrategy = AddressSelectorMode.HashAlgorithm)]
        Task<bool> EvalMessage(string flightId, EvalMessage evalMessage);
    }
```

## 实现应用接口
WS服务的应用类与一般业务应用类不同的是,继承的基类是`WSServiceBase`, `OnOpen()`会发生在客户端与Ws服务建立链接时触发(即：握手阶段),`OnClose()`会方法在客户端与服务端失去连接时触发。
一般地，我们会在客户端与服务握手阶段，将客户端的标识与`SessionId`的对应关系保存在一个字典中，以方便确定推送到哪个客户端。


```csharp
 [ModuleName("MessageNotify")]
    public class MessageNotify : WSServiceBase, IMessageNotify
    {
        private readonly ILogger<MessageNotify> _logger;
        private static readonly ConcurrentDictionary<string, string> _flightIds = new ConcurrentDictionary<string, string>();

        public MessageNotify(ILogger<MessageNotify> logger)
        {
            _logger = logger;
        }

        // 注意：ws服务之间的调用只能通过基于routepath远程调用，不支持通过接口创建代理远程调用
        public Task<bool> EvalMessage(string flightId, EvalMessage evalMessage)
        {
            var sessionId = _flightIds[flightId];
            if (string.IsNullOrEmpty(sessionId))
            {
                if (_logger.IsEnabled(Microsoft.Extensions.Logging.LogLevel.Debug))
                    _logger.LogDebug($"未建立与${flightId}的会话");
                return Task.FromResult(false);
            }
            GetClient().SendTo(evalMessage.Message, sessionId);
            return Task.FromResult(true);
        }

        protected override void OnOpen()
        {
            var flightId = Context.QueryString["flightId"];
            if (!string.IsNullOrEmpty(flightId))
            {
                _flightIds[flightId] = ID;
            }
        }
    }
```

## 推送消息
需要注意的是, ws服务之间的调用只能通过基于routepath远程调用，不支持通过接口创建代理远程调用。
一般情况下，建议在通用的业务组件，创建一个调用ws的服务，这样方便各个业务组件的消息推送。

```csharp
public class NotifyMessageProxy : ITransientDependency
{
    private const string NotifyApi = "v1/messagenotify//evalmessage";
    
    public Task<bool> NotifyEvalMessage(string flightId, string message)
    {
        var rpcParams = new Dictionary<string, object>();
        rpcParams.Add("flightId", flightId);
        rpcParams.Add("evalMessage", new
        {
            Message = message
        });
        return ServiceLocator.GetService<IServiceProxyProvider>()
                    .Invoke<bool>(rpcParams, NotifyApi, "MessageNotify");
    }
}
```

