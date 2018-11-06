---
title: 微服务框架Srcc之事件总线(发布-订阅)
date: 2018-11-06 20:07:31
categories: "微服务"
tags:
  - 微服务
  - surging
  - Srcc
  - 开源框架
---

# 消息中间件
SRCC服务引擎扩展了基于eventbus的rabbitmq和kafka事件总线，组件可以选择绑定 Normal，Retry(Dead letter)，Fail ，如下图所示:

![eventbus](evnetbus.png)

# 使用场景
## 流量削峰
比较典型的案例是:商品秒杀和抢购, 购买/秒杀是如今很常见的一个应用场景，在高并发的流量访问下可以将用户放入到抢购队列中，购买成功则销毁消息。

## 最终数据的一致性
在大型业务中，系统一般由多个独立的服务组成，在分布式调用时候把消息放入到rabbitmq 队列中，再通过消息的幂等性来解决数据的最终一致性

## 订单失效处理
在购买商品/服务生成订单业务中，会设定支付时间，如果一直未支付，会直接关闭订单，而这个场景可以通过死信队列的来解决

# 用法
1. 通过`srccsetting.json`的配置文件的`Srcc.Packages`节点指定使用的消息中间件的组件包,如果使用的是RabbitMq则指定`EventBusRabbitMQModule`模块,使用的是`EventBusKafkaModule`模块,并将相应的组件包安装到应用层。

2. 通过`eventBusSettings.json`对使用的消息中间件进行配置
如果使用的是RabbitMq消息中间件，则配置的案例为:
```json
{
  "EventBusConnection": "${EventBusConnection}|127.0.0.1",
  "EventBusUserName": "${EventBusUserName}|guest", //用户名
  "EventBusPassword": "${EventBusPassword}|guest", //密码
  "VirtualHost": "${VirtualHost}|/",
  "MessageTTL": "${MessageTTL}|30000", //消息过期时间，比如过期时间是30分钟就是1800000
  "RetryCount": "${RetryCount}|1", //重试次数，这里设置的延迟队列，只能设置为1
  "FailCount": "${FailCount}|3", //处理失败流程重试次数，如果出现异常，会进行重试
  "PrefetchCount": "${PrefetchCount}|0", //设置均匀分配消费者消息的个数
  "BrokerName": "srcc_demo",
  "Port": "${EventBusPort}|5672"
}
```

如果使用的Kafka消息中间件，则配置的案例为：

```json
 {
    "Servers": "${EventBusConnection}|127.0.0.1:9092",
    "MaxQueueBuffering": "${MaxQueueBuffering}|10",
    "MaxSocketBlocking": "${MaxSocketBlocking}|10",
    "EnableAutoCommit": "${EnableAutoCommit}|false",
    "LogConnectionClose": "${LogConnectionClose}|false",
    "OffsetReset": "${OffsetReset}|earliest",
    "GroupID": "${EventBusGroupID}|srccdemo"
  }
```

3. 事件处理器是通过继承`BaseIntegrationEventHandler`或者`IIntegrationEventHandler`实现的，再通过`QueueConsumer`特性进行标识

```csharp
[QueueConsumer("UserLoginDateChangeHandler",QueueConsumerMode.Normal)]
    public  class UserLoginDateChangeHandler : BaseIntegrationEventHandler<UserEvent>
    {
        private readonly IUserService _userService;
        public UserLoginDateChangeHandler()
        {
            _userService = ServiceLocator.GetService<IUserService>("User");
        }
        public override async Task Handle(UserEvent @event)
        {
            Console.WriteLine($"消费1。");
            await _userService.Update(@event.UserId, new UserModel()
            {
                Age = @event.Age,
                Name = @event.Name,
                UserId = @event.UserId
            });
            Console.WriteLine($"消费1失败。");
            throw new Exception();
        }

        public override Task Handled(EventContext context)
        {
            Console.WriteLine($"调用{context.Count}次。类型:{context.Type}");
            var model = context.Content as UserEvent;
            return Task.CompletedTask;
        }
    }
```

4. 消息时通过事件总线`IEventBus` 的 `Publish`方法发布的。

```csharp
public void Publish(IntegrationEvent @event)
{
    GetService<IEventBus>().Publish(@event);
}
```