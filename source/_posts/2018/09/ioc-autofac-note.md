---
title: Ioc容器之Autofac
date: 2018-09-26 22:25:48
categories: "ioc容器"
tags:
- Ioc
- 依赖注入
- 开源组件
---

## 术语

### 控制反转
控制反转(Inversion of Control，缩写为`IoC`)，面向对象编程中的一种设计原则，用来减低计算机代码之间的耦合度。借助于"第三方[即IOC容器]"实现具有依赖关系的对象之间的解耦(责任的反转)。

通过引入第三方的Ioc容器，使得对象与对象之间没有了耦合性，全部对象的控制权全部上缴给第三方的`IOC容器`，所以，Ioc容器成为了真个系统的关键核心，它起到了一种类似“粘合剂”的作用，把系统中的所有对象粘合在一起发挥作用，如果没有这个“粘合剂”，对象与对象之间会彼此失去联系，这就是有人把IOC容器比喻成“粘合剂”的由来。

- 控制反转名称的由来?
对象A获得依赖对象B的过程,由主动行为变为了被动行为，控制权颠倒过来了，这就是“控制反转”这个名称的由来（对象的创建与生命周期的维护的控制权转移）。

#### 实现方法
1. 依赖注入
  - 构造注入
  - 属性注入
  - 通过注解

2. 依赖查找

#### Ioc容器
Ioc容器： 控制反转中引入的第三方对象，通过Ioc容器将对象与对象的关系进行解耦，对象的创建与维护让渡给第三方容器。
Ioc容器负责维护对象与对象之间的关系，并负责对象的创建和对象生命周期的维护。

### 依赖注入
依赖注入就是将实例变量传入到一个对象中去

#### 什么是依赖?
在Class A 中，有 Class B的实例，则称Class A 对 Class B 有一个依赖。

例如:
```csharp
public class Human {
    ...
    Father father;
    ...
    public Human() {
        father = new Father();
    }
}
```

> **Notes:** `Human`中用到一个`Father`对象，我们就说类`Human`对类`Father`有一个依赖。

#### 控制反转和依赖注入的关系
- `控制反转`是一种思想
- `依赖注入`是一种设计模式

### 依赖倒置
依赖倒置(Dependence Inversion Principle，缩写为`DIP`)，是一种设计原则，是指: 
- 高层模块不应该依赖底层模块，都应该依赖于抽象
- 抽象不应该依赖于具体，具体依赖于抽象

原因:
- 若高层依赖于底层，那么底层的变动也会导致高层的变动，这就会导致模块的复用性降低而且大大提高了开发的成本。
- 若是依赖于抽象的话，那么是比较稳定的，底层或者高层的变动都不会互相影响

#### 低层模块
不可分割的原子逻辑，可能会根据业务逻辑经常变化。

#### 高层模块
低层模块的再组合，对低层模块的抽象。

#### 抽象
接口或抽象类（是底层模块的抽象，特点：不能直接被实例化）


## Autofac简介

`Autcofac` 是一个开源的Ioc框架。

将`Autofac`整合到应用的基本模式如下:
- 按照 控制反转 (`IoC`) 的思想构建你的应用.
- 添加`Autofac`引用.
- 在应用的 `startup` 处...
- 创建 `ContainerBuilder`.
- 注册组件.
- 创建 **容器** ,将其保存以备后续使用.
- 应用执行阶段...
- 从容器中创建一个生命周期.
- 在此生命周期作用域内解析组件实例.

## 注册组建和构建Ioc容器
Autofac 通过`ContainerBuilder`来注册组建并构建`Ioc容器`。
通过 `ContainerBuilder`对象的`Build()`方法构建Ioc容器。

### 组件和服务

组件可以通过 `反射` (注册指定的.NET类或开放结构的泛型)创建; 通过提供现成的`实例`(你已创建的一个对象实例)创建; 或者通过`lambda 表达式` (一个执行实例化对象的匿名方法)来创建. 

注册 组件 时, 我们得告诉Autofac, 组件暴露了哪些 服务， 默认地, 类型注册时大部分情况下暴露它们自身:

```csharp
// Create the builder with which components/services are registered.
var builder = new ContainerBuilder();

// Register types that expose interfaces...
builder.RegisterType<ConsoleLogger>().As<ILogger>();

// Register instances of objects you create...
var output = new StringWriter();
builder.RegisterInstance(output).As<TextWriter>();

// Register expressions that execute to create objects...
builder.Register(c => new ConfigReader("mysection")).As<IConfigReader>();

// Build the container to finalize registrations
// and prepare for object resolution.
var container = builder.Build();

// Now you can resolve services using Autofac. For example,
// this line will execute the lambda expression registered
// to the IConfigReader service.
using(var scope = container.BeginLifetimeScope())
{
  var reader = scope.Resolve<IConfigReader>();
}
```

### 注册组件的方法
#### 反射组件
  - 通过类型注册
  - 指定构造函数
  
   ```chsarp
   builder.RegisterType<MyComponent>()
       .UsingConstructor(typeof(ILogger), typeof(IConfigReader));
   ```
#### 实例组件

#### Lambda表达式组件

```csharp
builder.Register(c => new A(c.Resolve<B>()));
```
表达式提供的参数 `c` 是 组件上下文 (一个 `IComponentContext` 对象) , 组件在其中被创建. 你可以使用这个参数来从容器中解析出其他值来帮助创建你的组件. 使用这个参数而不是一个闭包来访问容器非常重要 这样可以保证 对象精确的释放 并且可以很好的支持嵌套容器.
- 复杂参数
- 参数注入
- 通过参数值选择具体的实现
```csharp
builder.Register<CreditCard>(
  (c, p) =>
    {
      var accountId = p.Named<string>("accountId");
      if (accountId.StartsWith("9"))
      {
        return new GoldCard(accountId);
      }
      else
      {
        return new StandardCard(accountId);
      }
    });
```



#### 泛型注入
- 通过`RegisterGeneric()` 方法

```csharp
builder.RegisterGeneric(typeof(NHibernateRepository<>))
       .As(typeof(IRepository<>))
       .InstancePerLifetimeScope();
```



## 解析服务

在注册完组件并暴露相应的服务后, 你可以从创建的容器或其子 生命周期 中解析服务. 让我们使用 `Resolve()` 方法来实现:

```csharp
var builder = new ContainerBuilder();
builder.RegisterType<MyComponent>().As<IService>();
var container = builder.Build();

using(var scope = container.BeginLifetimeScope())
{
  var service = scope.Resolve<IService>();
}
```

> **Notes:** 有时在我们的应用中也许可以从根容器中解析组件, 然而这么做有可能会导致 **内存泄漏**. 推荐你 **总是从生命周期中解析组件**, 以确保服务实例被妥善地释放和垃圾回收。

解析服务时, Autofac 自动链接起服务所需的整个依赖链上不同层级并解析所有的依赖来完整地构建服务. 如果你有处理不当的 循环依赖 或缺少了必需的依赖, 你将得到一个 `DependencyResolutionException`.

如果你不清楚一个服务是否被注册了, 你可以通过 `ResolveOptional()` 或 `TryResolve()` 尝试解析:

```csharp
// If IService is registered, it will be resolved; if
// it isn't registered, the return value will be null.
var service = scope.ResolveOptional<IService>();

// If IProvider is registered, the provider variable
// will hold the value; otherwise you can take some
// other action.
IProvider provider = null;
if(scope.TryResolve<IProvider>(out provider))
{
  // Do something with the resolved provider value.
}
```


## 控制作用域和生命周期
### 服务的生命周期
服务的 **生命周期** 是指服务实例在你的应用中存在的时长 - 从开始实例化到最后 释放 结束

### 服务的作用域
服务的 **作用域** 是指它在应用中能共享给其他组件并被消费的作用域。
- 在你的应用中你有个全局的静态单例 - 该全局对象实例的 "作用域" 将会是整个应用. 另一方面, 如果你在一个 for 循环中创建了引用了全局单例的一个局部变量 - 那么这个局部变量就拥有比全局变量小很多的作用域

### 生命周期作用域 
生命周期作用域等同于你应用中的一个工作单元. 一个工作单元将会在开始时启动生命周期作用域, 然后需要该工作单元的服务被从生命周期作用域中解析出. 当你解析服务时, Autofac将会追踪被解析的可释放/可销毁 (`IDisposable`) 组件. 在工作单元最后, 你释放了相关的生命周期作用域然后Autofac将会自动清理/释放那些被解析的服务.生命周期作用域等同于你应用中的一个工作单元. 一个工作单元将会在开始时启动生命周期作用域, 然后需要该工作单元的服务被从生命周期作用域中解析出. 当你解析服务时, Autofac将会追踪被解析的可释放/可销毁 (`IDisposable`) 组件. 在工作单元最后, 你释放了相关的生命周期作用域然后Autofac将会自动清理/释放那些被解析的服务.

在你的应用中, 最好记住以下概念这样就能有效使用你的资源:
- **永远从一个生命周期作用域而不是从根容器中解析服务.** 由于生命周期作用域有追踪可释放资源的性质, 如果你从一个容器 ("根生命周期作用域") 中解析了太多组件, 无意间也许你就会造成内存泄露. 根生命周期会在它存在的时间 (通常是应用的生命周期) 内保持住可释放组件因此它也能释放它们. 你可以选择性的改变释放行为, 但从作用域内解析是个良好的实践. 如果Autofac检测到使用单例或共享组件, 它会自动把它们安放在一个合适的追踪作用域之内


### 生命周期事件
Autofac暴露了一些能在实例生命周期多个阶段拦截到的事件. 这些事件在组件注册时被订阅 (或者也可以通过附加到 `IComponentRegistration` 接口
#### 激活时
```csharp
builder.RegisterType<TConcrete>() // FAILS: will throw at cast of TInterfaceSubclass
       .As<TInterface>()          // to type TConcrete
       .OnActivating(e => e.ReplaceInstance(new TInterfaceSubclass()));
```
#### 激活后

#### 释放时

## 注册Named命名和Key Service服务

### 注册命名服务
可以通过一个字符串名字要标示服务，这时我们用`Named`代替`As`：

```csharp
builder.Register<OnlineState>().Named<IDeviceState>("online");
```

解析:
```csharp
var r = container.ResolveNamed<IDeviceState>("online");
```

### 注册Key Service服务

假设存在设备状态枚举:
```csharp
public enum DeviceState { Online, Offline }
```

枚举每一个值分别对应一个具体的实现类:

```csharp
public class OnlineState : IDeviceState { }
public class OfflineState : IDeviceState { }

```

每一个枚举值对应的组件分别注册到容器中:
```csharp
var builder = new ContainerBuilder();
builder.RegisterType<OnlineState>().Keyed<IDeviceState>(DeviceState.Online);
builder.RegisterType<OfflineState>().Keyed<IDeviceState>(DeviceState.Offline);
```

解析其中一种状态对应的组件：
```csharp
var r = container.ResolveKeyed<IDeviceState>(DeviceState.Online);
```