---
title: Ioc容器之Ninject
date: 2018-09-29 21:07:20
categories: "ioc容器"
tags:
  - Ioc
  - 依赖注入
  - 开源组件
---

## 简介

`Ninject`是一个快如闪电、超轻量级的基于.Net平台的`IOC容器`，主要用来解决程序中模块的耦合问题，它的 **目的在于做到最少配置**。 如果不喜欢配置，不喜欢重量级的IOC框架，在这种场景下就可以选择`Ninject`框架。

## Ninject用法

1. 使用`Ninject`需要通过nuget工具安装`Ninject`包

2. 继承`Ninject.Modules.NinjectModule`,实现自己的模块,并通过重写`Load()`方法实现依赖注入(`DI`)。
  ```csharp
  public class MyModule : Ninject.Modules.NinjectModule
  {
      public override void Load()
      {
          Bind<ILogger>().To<FileLogger>();
          Bind<ILogger>().To<DBLogger>();
      }
  }
  ```
3. 通过`StandardKernel`加载模块和解析对象。
  ```csharp
  private static IKernel kernel = new StandardKernel(new MyModule());

  static void Main(string[] args)
  {
      ILogger logger = kernel.Get<ILogger>();//获取的是FileLogger
      logger.Write(' Hello Ninject!');
      Console.Read();
  }
  ```

### Ninject常用方法属性说明
1. Bind<T1>().To<T2>()
 就是接口`IKernel`的方法，把某个类绑定到某个接口，`T1`代表的就是接口或者抽象类，而`T2`代表的就是其实现类。

2. Get<ISomeInterface>()
 获取某个接口的实例
3. Bind<T1>().To<T2>().WithPropertyValue('SomeProprity', value);
 在绑定接口的实例时，同时给实例NinjectTester的属性赋值
```chsarp
ninjectKernel.Bind<ITester>().To<NinjectTester>().WithPropertyValue('_Message', '这是一个属性值注入');
```
4. Bind<T1>().To<T2>(). WithConstructorArgument('someParam', value);
 为实例的构造方法所用的参数赋值
5. Bind<T1>().ToConstant()
 意思是绑定到某个已经存在的常量
6. Bind<T1>().ToSelf()
 该方法是绑定到自身，但是这个绑定的对象只能是具体类，不能是抽象类。为什么要自身绑定呢？其实也就是为了能够利用Ninject解析对象本身而已。
 ```csharp
  ninjectKernel.Bind<StudentRepository>().ToSelf();
  StudentRepository sr = ninjectKernel.Get<StudentRepository>();
 ```

7. Bind<T1>().To<T2>().WhenInjectedInto<instance>()
  这个方法是条件绑定，就是说只有当注入的对象是某个对象的实例时才会将绑定的接口进行实例化

8. Bind<T1>().To<T2>().InTransientScope()或者Bind<T1>().To<T2>().InSingletonScope()
  为绑定的对象指明生命周期

9. Load()方法
  `Load()`方法其实是抽象类`Ninject.Modules.NinjectModule`的一个抽象方法，通过重写`Load()`方法可以对相关接口和类进行集中绑定。

10. Inject属性
  可以通过在构造函数、属性和字段上加 Inject特性指定注入的属性、方法和字段等，如果没有在构造函数上指定`Inject`特性，则默认选择第一个构造函数
  
```csharp
public class MessageDB: IMessage
{
  public MessageDB()
  { }

  public MessageDB(object msg)
  {

     Console.WriteLine('使用了object 参数构造：{0}',msg);
  }


  [Inject]
  public MessageDB(int msg)
  {
   Console.WriteLine('使用了int参数构造：{0}',msg);
  }

  public string GetMsgNumber()
  {
    return '从数据中读取消息号!';
  }
}
```

## ASP.NET MVC与Ninject
1. 继承DefaultControllerFactory，重载GetControllerInstance方法，实现自己的NinjectControllerFactory类
```csharp
public class NinjectControllerFactory : DefaultControllerFactory
{
    private IKernel ninjectKernel;
    public NinjectControllerFactory()
    {
        ninjectKernel = new StandardKernel();
        AddBindings();
    }
    protected override IController GetControllerInstance(RequestContext requestContext, Type controllerType)
    {
        return controllerType == null ? null : (IController)ninjectKernel.Get(controllerType);
    }
    private void AddBindings()
    {
        ninjectKernel.Bind<IStudentRepository>().To<StudentRepository>();
    }
}
```
2. 在函数`Application_Start()` 注册自己的控制器工厂类
```csharp
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        AreaRegistration.RegisterAllAreas();
        WebApiConfig.Register(GlobalConfiguration.Configuration);
        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        RouteConfig.RegisterRoutes(RouteTable.Routes);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
        AuthConfig.RegisterAuth();

       //设置控制器工厂生产类为自己定义的NinjectControllerFactory
        ControllerBuilder.Current.SetControllerFactory(new NinjectControllerFactory());
    }
}
```

3. 最后添加控制器，并注入依赖代码

## Asp.net Core 与Ninject

## 参考资料

- [ASP.NET MVC IOC 之Ninject攻略](https://blog.csdn.net/csethcrm/article/details/44854069)