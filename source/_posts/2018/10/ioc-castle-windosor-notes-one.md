---
title: Ioc容器之Castle Windsor
date: 2018-10-02 00:23:04
categories: "ioc容器"
tags: 
- Ioc
- 依赖注入
- 开源组件
---

## 概念
### Ioc和Ioc容器的区别
`IoC` 是一个框架使用的设计原则，用来让开发者来扩展框架或使用它创建应用程序。其基本思路是，框架是知道程序员的对象，并对它们进行调用。
`IoC容器`使用上面所述的（简言之）原则管理类。包括，它们的创建，销毁，生命期，配置和依赖关系。这样类并不需要获取并配置它们所依赖的类。这在系统中极大地减少了耦合，并且简化了重用和可测试性。

> **Notes:**
> `Ioc` 与 `Ioc容器`不是同一个概念。
> 那些认为`IoC`与`IoC容器`是一个同义词的人导致了一些混乱。如前所述，IoC是一个更广泛的原则。

### 服务、组件、依赖
![服务,组件,依赖](http://static.bookstack.cn/projects/Windsor-doc-cn/docs/images/component.png)

### 服务
`服务`是高等级的。它定义了某个功能、协议所需要的方面，但它不泄漏关于谁、如何、哪里准备了某个功能的任何细节。
一般地，服务会被定义为`接口`。

### 组件
`组件`和`服务`有关。`服务`是一个抽象的术语，但我们处理具体，真实的世界。`组件`一般是指实现了某个服务的具体的类。

### 依赖
一个组件工作来完成它的服务不是空谈,一个组件完成某项工作（功能）可能会需要其他组件的服务协助，也就是说某个功能可能需要多个组件协同工作（例如: 咖啡店依赖于公用事业公司（电力）提供的服务、他的供应商（以得到豆类，牛奶等），大多数组件将最终委托它们的非必须方面给其他组件）。

### 组件的创建过程
![组件的创建过程](http://static.bookstack.cn/projects/Windsor-doc-cn/docs/images/creation-flow.png)

1. 定位处理器(Locating Handler)
- 它调用与它相关联的所有`ComponentResolvingDelegate`，让他们有机会在实际开始之前来影响它的决定。这里有个例子， 什么时候委托传递到Fluent注册API的`DynamicParameters`方法。（Fluent注册API）
- 如果未提供内嵌参数，它会检查该组件及其所有强制依赖是否能够解析。如果不能，抛出`HandlerException`异常。
- 最后，处理器要求生命期方式管理器解析该组件。

2. 生命期方式管理器
有一个它可以重复使用的组件实例，它获得并直接返回该实例返回到处理器。如果没有，它要求它的`组件激活器`创建一个。

3. 组件激活器
组件激活器负责创建组件的实例。各种激活器有各自的实现。当您通过`UsingFactoryMethod`创建组件时，您提供的委托将被调用以创建实例。工厂支持设施或远程设施有它们自己的一套激活器，用于执行组件的自定义初始化。

4. 发布政策
`处理器`调用`发布政策`以跟踪组件，然后将组件传递给容器，容器将组件返回给用户
多数时候你应该使用 `DefaultComponentActivator`

### 依赖是如何解析的
Windsor 中的组件很少是独立的。毕竟，Windsor 最主要的任务是查找和管理依赖。

#### 依赖解析器（Dependency Resolver）
Windsor 使用依赖解析器（实现`IDependencyResolver`接口的类型）解析组件的依赖。默认依赖解析器（`DefaultDependencyResolver` 类）。

1. 创建上下文（Creation Context）
首先依赖解析器为依赖调用`CreationContext`。创建上下文
  - 首先尝试用名称解析依赖（传递给 `Resolve` 的参数: `container.Resolve<IFoo>(new Arguments(new { inlineDependency = "its value" }));`）
  - 然后通过使用内联提供的依赖的类型(通过 Fluent 注册 API 的方法 `DynamicParameters`传递的参数)。
2. 没有内联参数能够满足依赖，解析器询问处理器是否能够满足
  - 首先尝试通过名字解析依赖
  - 然后通过类型
3. 上面那些地方都不能解析依赖，解析器询问它的每一个`子解析器`（实现(`ISubDependencyResolver`)）是否能够提供依赖

4. 上面那些都不能解析依赖，容器尝试自己解析。根据依赖的类型，容器尝试下面的步骤
  - 对于参数依赖，容器尝试检查 `ComponentModel`的 `Parameters` 集合以匹配依赖。包括 XML 提供的参数，或通过 Fluent API 的 `Parameters` 方法传递的参数。
  - 对于 `service override dependency`，容器尝试通过指定关键字匹配组件。
  - 对于服务依赖，容器将会尝试通过指定关键字匹配任意一个组件
5. 如果上面都不行，抛出`DependencyResolverException`异常

## 使用容器
### 三个容器调用模式
IoC 框架与其他框架不同之处在于你不会在代码中看见许多对框架的调用，在大多数应用中（忽略它们的大小和复杂性）你仅在三个地方直接调用容器。
该模式被称为 `Three Calls`。有时被称为 `RRR` - `注册`，`解析`，`释放`（`Register`, `Resolve`, `Release`）。在应用中应用容器只需要在三个地方调用容器。更精确的说，在`入口项目`。

使用容器的一个非常重要的方面-在应用中有一个应用实例。你只需要一个单独的，容器的`根容器`实例。在一些非常罕见的高级场景，你可能会创建“子”容器，但是它们都与那个单一的无处不在的根容器栓在一起，作为`子容器`。

### Call one - 引导程序（bootstrapper）
在引导程序中创建和配置容器。 例如:
```csharp
public IWindsorContainer BootstrapContainer()
{
    return new WindsorContainer()
        .Install(
            Configuration.FromAppConfig(),
            FromAssembly.This()
            //perhaps pass other installers here if needed
        );
}
```
在引导程序中需要做的事:
1. 创建将要使用的容器。
2. 需要的话自定义容器( 大多数时候你不会这样做，通常是从不)。
3. 注册所有容器将要管理的所有组件。即上面代码中 `Install` 方法的调用。这里你传递 封装有关应用程序的特定组件的所有信息的安装器 。这就是你将要看到的大部分工作。

### Call two - Resolve
显式从容器解析的全部组件。容器然后构造它们的依赖和依赖的依赖等等的整个图。

```csharp
var container = BootstrapContainer();
var shell = container.Resolve<IShell>();
shell.Display();
```
### Call three - Dispose

容器管理组件的整个生命期，并且在关闭应用之前我们需要关闭容器，这将反过来停用它管理的所有组件（比如回收它们）。这就是为什么在关闭应用之前调用`container.Dispose()`是如此重要。

## Windsor 安装器（Installers）
### 简介
Windsor使用`安装器` (实现 `IWindsorInstaller` 接口的类型) 封装和分离你的注册逻辑，一些帮助类，比如 `Configuration` 和`FromAssembly`使得安装工作轻而易举。

### IWindsorInstaller 接口
安装器是实现了`IWindsorInstaller`接口的简单类型。 该接口只有一个`Install`方法。该方法获得容器的实例，然后就可以使用`Fluent注册`API注册组件。

```csharp
public class RepositoriesInstaller : IWindsorInstaller
{
   public void Install(IWindsorContainer container, IConfigurationStore store)
   {
      container.Register(AllTypes.FromAssemblyNamed("Acme.Crm.Data")
                            .Where(type => type.Name.EndsWith("Repository"))
                            .WithService.DefaultInterfaces()
                            .Configure(c => c.LifeStyle.PerWebRequest));
   }
}
```
- 分离安装器: 通常一个单独的安装器安装相关服务的连续闭集（比如仓储，控制器，等等），每个集合都有一个独立的安装器。

### 使用安装器
在创建安装器之后，你必须在启动程序中将它们安装到容器。为此，使用容器的`Install`方法

```csharp
var container = new WindsorContainer();
container.Install(
   new ControllersInstaller(),
   new RepositoriesInstaller(),
   // and all your other installers
);
```

### FromAssembly帮助类
通过使用`FromAssembly` 类，将事情留给 Windsor 完成。这个类有一些选择一个或多个程序集的方法，然后它会实例化那些程序集中的所有安装器类型。这样做的好处是，当你为这些程序集添加新的安装器时，它们会被 Windsor 自动选中，你不需要做额外的工作。

```csharp
container.Install(
   FromAssembly.This(),
   FromAssembly.Named("Acme.Crm.Bootstrap"),
   FromAssembly.Containing<ServicesInstaller>(),
   FromAssembly.InDirectory(new AssemblyFilter("Extensions")),
   FromAssembly.Instance(this.GetPluginAssembly()));
```

> **Notes**
> - 在使用`FromAssembly`时，你不应该依赖安装器将会被实例化或安装的顺序。它是不确定的，意味着你不会知道执行的顺序。
> - 如果你需要以指定顺序安装安装器，使用 `InstallerFactor`

### InstallerFactory帮助类
如果需要更严格的控制程序集的安装器（影响安装的顺序，改变实例化方式或只安装一部分，而不是全部），你可以从这个类派生，并提供你自己的实现去实现这些目标。

### Configuration 类
可以用它来访问`AppDomain`配置文件（的`app.config`或`web.config`文件）中的配置，或任意的XML文件
```csharp
container.Install(
   Configuration.FromAppConfig(),
   Configuration.FromXmlFile("settings.xml"),
   Configuration.FromXml(new AssemblyResource("assembly://Acme.Crm.Data/Configuration/services.xml"))
);
```
## 注册 API 引用
注册 API 提供注册和配置组件的`Fluent方法`。这是推荐的注册方式，比过细的XML注册更好。因为它是强类型的且容易被重构，它可以作为XML配置的替代方式或补充。和XML一起使用更为有利。
1. 一个一个的注册组件
2. 按照约定注册组件
3. 使用XML配置

## AOP，代理，和拦截器
Windsor 可以通过使用拦截器`DynamicProxy`实现切面编程。

### 如何创建代理
不需要显式指定希望一个组件成为代理。Windsor 将会自动创建代理，如果下面的任何一个条件被满足：
1. 有为该组件指定的`拦截器`（即包含通过组件拦截器选择器动态选择的拦截器）。
2. 有为该组件指定的 `mixins`
3. 有为该组件指定的其他接口

### 指定拦截器

1. 使用 Fluent 注册 API
2. 使用 XML 配置
3. 使用 Attributes (just for interceptors) -- `InterceptorAttribute`

#### 使用InterceptorAttribute需要注意的事项
```csharp
public interface IOrderRepository
{
   Order GetOrder(Guid id);
}
[Interceptor("cache")]
[Interceptor(typeof(LoggingInterceptor))]
public class OrderRepository: IOrderRepository
{
   Order GetOrder(Guid id)
   {
      // some implementation
   }
}
```

- 将特性放在组件类上，而不是接口。
- 可以通过类上面的多个特性指定多个拦截器。
- 可以通过类型或名称指定拦截器。

### 使用拦截器
```csharp
container.Register(
   Component.For<LoggingInterceptor>().Lifestyle.Transient,
   Component.For<CacheInterceptor>().Lifestyle.Transient.Named("cache"),
   Component.For<IOrderRepository>().ImplementedBy<OrderRepository>());
```

> **notes:**
> 拦截器应该是临时的： 强烈建议总是让拦截器是临时的。因为拦截器可以拦截多个使用不同生命期方式的组件，最好是让拦截器自身的寿命不超过它们拦截的组件。因此除非有充足的理由不这么做，总是让拦截器是临时的

## 组件的生命周期

### 生命周期类型
#### 标准生命期类型: 常见类型
1. 单例（Singleton）
单例组件在一个容器中只有一个实例。实例将在第一次请求时被创建，随后每次需要时都进行重用。显式释放单例（通过调用`container.Release(mySingleton)`）什么都不会做。这个实例将会在它注册的容器销毁时释放。
> **Notes:**
> 单例是默认的生命期类型，将会在未显式指定的时候使用

2. 瞬态(Transient)
`Transient组件`不会绑定到任何作用域。每次需要`Transient组件`的实例时，容器都会产生一个新的实例，而不会重用它们。可以说`Transient组件`的作用域有它的使用者决定。因此当使用Transient组件的对象释放时，它们也被释放。

> **Notes:**
> - 为了确保正确的组件生命周期管理，Windsor 可能跟踪这些组件。 这意味着，除非你释放它们，否则垃圾收集器将无法收回它们，这最终导致事实上的内存泄漏,记住`释放`那些你显式`解析`的。
> - 当你想控制组件的生命期时，`transient`是一个好选择。当你每次都需要全新的实例时。当然 `transient组件`不需要是线程安全的，除非你将在多线程条件下使用。在大多数程序中你会发现你大多数的组件最后都是`transient`的。

3. 每次Web请求（PerWebRequest）
组件的实例会在一个单一的Web请求范围内共享。该实例将在Web请求的范围第一次请求时创建。显式释放它是无效的。实例将在web请求结束时释放。

> **Notes:**
> - 注册`PerWebRequestLifestyleModule`: 为了`PerWebRequest`的正常运行需要将一个`IHttpModule` - `Castle.MicroKernel.Lifestyle.PerWebRequestLifestyleModule`注册到 `web.config` 文件中:

```xml
<httpModules>
   <add name="PerRequestLifestyle" type="Castle.MicroKernel.Lifestyle.PerWebRequestLifestyleModule, Castle.Windsor"/>
</httpModules>
```

#### 标准生命期类型: 不常见类型
4. 范围（Scoped）

5. Bound
如果默认选项不能满足你的需求，你可以提供自定义方式以选择要绑定的组件，使用下面`BoundTo`方法的重载。


### 生命周期
![生命周期](http://static.bookstack.cn/projects/Windsor-doc-cn/docs/images/lifecycle-simplified.png)
Windsor 是一个控制组件创建和销毁的工具。

1. 创建 - 所有的事情都在`container.Resolve` 或类似的方法内发生 (查看组件是如何创建的以了解详情)。
2. 使用 - 在你的代码中使用组件完成工作。
3. 销毁 - 所有的事情在`container.ReleaseComponent`之内/或之后或组件的生命期范围结束时发生。

## 扩展容器

### 设施(Facilities)
设施是扩展容器的主要方式。使用设施 **可以将外部框架与容器集成**，比如`WCF` 或 `NHibernate`，为容器添加新的功能，比如事件连接（event wiring），事务支持（transaction support）……，或为组件（synchronization, startable semantics…）

### 如何使用设施
在使用设施前需要将其注册到容器，通过代码（如下所示）或者XML 配置。
```csharp
container.AddFacility<TypedFactoryFacility>();
```
