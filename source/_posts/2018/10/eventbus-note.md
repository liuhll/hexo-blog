---
title: 事件总线
date: 2018-10-04 20:01:24
categories: "微服务"
tags:
- 事件总线
- 微服务
---

## 简介
`事件总线`是对 **发布-订阅模式** 的一种实现，他是一种集中式事件处理机制，允许不同的组件之间进行彼此通信而又不需要互相依赖，达到一种解耦的目的。

![事件总线](https://upload-images.jianshu.io/upload_images/2799767-6f44bdefa88a23a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 事件本质
`事件`是由`事件源`和`事件处理`组成,是对消息的封装，是IDE编程环境为了简化编程而提供的有用的工具。

###  发布订阅模式

#### 定义
定义对象间一种 **一对多**的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并被自动更新。

#### 角色
- **发布方（Publisher）**: 也称为`被观察者`，当状态改变时负责通知所有订阅者。
- **订阅方（Subscriber）**：也称为`观察者`，订阅事件并对接收到的事件进行处理。

#### 实现方式
1. 简单的实现方式：由`Publisher`维护一个订阅者列表，当状态改变时循环遍历列表通知订阅者。
2. 委托的实现方式：由`Publisher`定义事件委托，`Subscriber`实现委托。

发布订阅模式中有两个关键字，**通知和更新**。 被观察者状态改变通知观察者做出相应更新。 解决的是当对象改变时需要通知其他对象做出相应改变的问题。

![发布-订阅模式](https://upload-images.jianshu.io/upload_images/2799767-8a17f6e834278167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 实现事件总线

### 提取事件源
抽象一个接口`IEvent`，用于封装事件。可以是一个空的标识接口，也可以包括`EventSource`和 `EventTime`。

```csharp
/// <summary>
/// 定义事件源接口，所有的事件源都要实现该接口
/// </summary>
public interface IEvent
{
    /// <summary>
    /// 事件发生的时间
    /// </summary>
    DateTime EventTime { get; set; }

    /// <summary>
    /// 触发事件的对象
    /// </summary>
    object EventSource { get; set; }
}
```
### 提取事件处理器
需要抽象出一个`IEventHandler`,用于抽象事件处理器，包含一个`Handler`方法。
```csharp
/// <summary>
 /// 泛型事件处理器接口
 /// </summary>
 /// <typeparam name="TEventData"></typeparam>
 public interface IEventHandler<TEventData> : IEventHandler where TEventData : IEvent 
 {
     /// <summary>
     /// 事件处理器实现该方法来处理事件
     /// </summary>
     /// <param name="event"></param>
     void HandleEvent(TEvent eventData);
 }
```


### 实现事件总线
1. `事件总线`主要定义三个方法，**注册**、**取消注册**、**事件触发**。提供统一的事件注册、取消注册和触发接口
2. `EventBus`要接管所有事件的发布和订阅，那它则需要有一个容器来记录事件源和事件处理。那又如何触发呢？有了事件源，我们就自然能找到绑定的事件处理逻辑，通过反射触发,事件总线利用反射完成事件源与事件处理的初始化绑定。
3. `事件总线`采用单例设计模式，确保了事件中心的唯一入口。
4. `事件总线`维护一个事件源与事件处理的映射字典`_eventAndHandlerMapping`。

```csharp
/// <summary>
/// 事件总线
/// </summary>
public class EventBus
{
    public static EventBus Default => new EventBus();

    /// <summary>
    /// 定义线程安全集合
    /// </summary>
    private readonly ConcurrentDictionary<Type, List<Type>> _eventAndHandlerMapping;

    public EventBus()
    {
        _eventAndHandlerMapping = new ConcurrentDictionary<Type, List<Type>>();
        MapEventToHandler();
    }

    /// <summary>
    ///通过反射，将事件源与事件处理绑定
    /// </summary>
    private void MapEventToHandler()
    {
        Assembly assembly = Assembly.GetEntryAssembly();
        foreach (var type in assembly.GetTypes())
        {
            if (typeof(IEventHandler).IsAssignableFrom(type))//判断当前类型是否实现了IEventHandler接口
            {
                Type handlerInterface = type.GetInterface("IEventHandler`1");//获取该类实现的泛型接口
                if (handlerInterface != null)
                {
                    Type eventDataType = handlerInterface.GetGenericArguments()[0]; // 获取泛型接口指定的参数类型

                    if (_eventAndHandlerMapping.ContainsKey(eventDataType))
                    {
                        List<Type> handlerTypes = _eventAndHandlerMapping[eventDataType];
                        handlerTypes.Add(type);
                        _eventAndHandlerMapping[eventDataType] = handlerTypes;
                    }
                    else
                    {
                        var handlerTypes = new List<Type> { type };
                        _eventAndHandlerMapping[eventDataType] = handlerTypes;
                    }
                }
            }
        }
    }

    /// <summary>
    /// 手动绑定事件源与事件处理
    /// </summary>
    /// <typeparam name="TEventData"></typeparam>
    /// <param name="eventHandler"></param>
    public void Register<TEventData>(Type eventHandler)
    {
        List<Type> handlerTypes = _eventAndHandlerMapping[typeof(TEventData)];
        if (!handlerTypes.Contains(eventHandler))
        {
            handlerTypes.Add(eventHandler);
            _eventAndHandlerMapping[typeof(TEventData)] = handlerTypes;
        }
    }

    /// <summary>
    /// 手动解除事件源与事件处理的绑定
    /// </summary>
    /// <typeparam name="TEventData"></typeparam>
    /// <param name="eventHandler"></param>
    public void UnRegister<TEventData>(Type eventHandler)
    {
        List<Type> handlerTypes = _eventAndHandlerMapping[typeof(TEventData)];
        if (handlerTypes.Contains(eventHandler))
        {
            handlerTypes.Remove(eventHandler);
            _eventAndHandlerMapping[typeof(TEventData)] = handlerTypes;
        }
    }

    /// <summary>
    /// 根据事件源触发绑定的事件处理
    /// </summary>
    /// <typeparam name="TEventData"></typeparam>
    /// <param name="eventData"></param>
    public void Trigger<TEventData>(TEventData eventData) where TEventData : IEventData
    {
        List<Type> handlers = _eventAndHandlerMapping[eventData.GetType()];

        if (handlers != null && handlers.Count > 0)
        {
            foreach (var handler in handlers)
            {
                MethodInfo methodInfo = handler.GetMethod("HandleEvent");
                if (methodInfo != null)
                {
                    object obj = Activator.CreateInstance(handler);
                    methodInfo.Invoke(obj, new object[] { eventData });
                }
            }
        }
    }
}
```