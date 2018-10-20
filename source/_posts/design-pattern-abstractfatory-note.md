---
title: 设计模式抽象工厂模式
date: 2018-10-16 22:47:43
categories: "设计模式"
tags:
  - 抽象工厂模式
  - 创建型
---

## 动机(Motivate)
 在软件系统中，经常面临着*一系列相互依赖的对象*的创建工作：同时，由于需求的变化，往往存在更多系列对象的创建工作。如何应对这种变化？
## 意图(Intent)
提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

## 主要解决的问题
主要解决接口选择的问题。

## 结构图（Structure）
![抽象工厂](abstractfactory_pattern_uml_diagram.jpg)

## 模式的组成
1. **抽象产品类角色（AbstractProduct）**：为抽象工厂中相互依赖的每种产品定义抽象接口对象，也可以这样说，有几种产品，就要声明几个抽象角色，每一个抽象产品角色和一种具体的产品相匹配。
2. **具体产品类（ConcreteProduct）**：具体产品类实现了抽象产品类，是针对某个具体产品的实现的类型。
3. **抽象工厂类角色（Abstract Factory）**：定义了创建一组相互依赖的产品对象的接口操作，每种操作和每种产品一一对应。
4. **具体工厂类角色（ConcreteFactory）**：实现抽象类里面的所有抽象接口操作，可以创建某系列具体的产品，这些具体的产品是“抽象产品类角色”的子类。

## 代码实现
```csharp
/////////////////////////////////////////////////////////////
// 客户端调用
/////////////////////////////////////////////////////////////
/// <summary>
/// 下面以不同系列房屋的建造为例子演示抽象工厂模式
/// 因为每个人的喜好不一样，我喜欢欧式的，我弟弟就喜欢现代的
/// 客户端调用
/// </summary>
class Client
{
    static void Main(string[] args)
    {
       // 哥哥的欧式风格的房子
       AbstractFactory europeanFactory= new EuropeanFactory();
       europeanFactory.CreateRoof().Create();
       europeanFactory.CreateFloor().Create();
       europeanFactory.CreateWindow().Create();
       europeanFactory.CreateDoor().Create();    
       //弟弟的现代风格的房子
       AbstractFactory modernizationFactory = new ModernizationFactory();
       modernizationFactory.CreateRoof().Create();
       modernizationFactory.CreateFloor().Create();
       modernizationFactory.CreateWindow().Create();
       modernizationFactory.CreateDoor().Create();
       Console.Read();
      }
  }

/////////////////////////////////////////////////////////////
// 抽象工厂                                                
/////////////////////////////////////////////////////////////
/// <summary>
/// 抽象工厂类，提供创建不同类型房子的接口
/// </summary>
public abstract class AbstractFactory
 {
     // 抽象工厂提供创建一系列产品的接口，这里作为例子，只给出了房顶、地板、窗户和房门创建接口
     public abstract Roof CreateRoof();
     public abstract Floor CreateFloor();
     public abstract Window CreateWindow();
     public abstract Door CreateDoor();
 }


/////////////////////////////////////////////////////////////
// 具体工厂类
/////////////////////////////////////////////////////////////
/// <summary>
/// 欧式风格房子的工厂，负责创建欧式风格的房子
/// </summary>
public class EuropeanFactory : AbstractFactory
{
     // 制作欧式房顶
     public override Roof CreateRoof()
     {
          return new EuropeanRoof();
      }
     // 制作欧式地板
     public override Floor CreateFloor()
     {
         return new EuropeanFloor();
     }
    // 制作欧式窗户
    public override Window CreateWindow()
    {
         return new EuropeanWindow();
    }

      // 制作欧式房门
      public override Door CreateDoor()
      {
           return new EuropeanDoor();
       }
}

/// <summary>
/// 现在风格房子的工厂，负责创建现代风格的房子
/// </summary>
public class ModernizationFactory : AbstractFactory
{
     // 制作现代房顶
    public override Roof CreateRoof()
    {
         return new ModernizationRoof();
     }
    // 制作现代地板
   public override Floor CreateFloor()
   {
        return new ModernizationFloor();
   }
   // 制作现代窗户
  public override Window CreateWindow()
  {
       return new ModernizationWindow();
  }
  // 制作现代房门
 public override Door CreateDoor()
 {
       return new ModernizationDoor();
  }
}
 
/////////////////////////////////////////////////////////////
// 抽象产品类角色
/////////////////////////////////////////////////////////////
/// <summary>
/// 房顶抽象类，子类的房顶必须继承该类
/// </summary>
public abstract class Roof
 {
      /// <summary>
     /// 创建房顶
    /// </summary>
    public abstract void Create();
}

/// <summary>
/// 地板抽象类，子类的地板必须继承该类
/// </summary>
public abstract class Floor
 {
      /// <summary>
     /// 创建地板
    /// </summary>
   public abstract void Create();
}

/// <summary>
/// 窗户抽象类，子类的窗户必须继承该类
/// </summary>
 public abstract class Window
  {
      /// <summary>
     /// 创建窗户
    /// </summary>
    public abstract void Create();
 }

 /// <summary>
 /// 房门抽象类，子类的房门必须继承该类
 /// </summary>
 public abstract class Door
  {
      /// <summary>
     /// 创建房门
    /// </summary>
    public abstract void Create();
 }

/////////////////////////////////////////////////////////////
// 具体产品类
/////////////////////////////////////////////////////////////
/// <summary>
/// 欧式地板类
/// </summary>
public class EuropeanFloor : Floor
 {
    public override void Create()
   {
        Console.WriteLine("创建欧式的地板");
   }
}


 /// <summary>
 /// 欧式的房顶
 /// </summary>
 public class EuropeanRoof : Roof
  {
       public override void Create()
      {
          Console.WriteLine("创建欧式的房顶");
      }
  }


/// <summary>
///欧式的窗户
/// </summary>
public class EuropeanWindow : Window
 {
     public override void Create()
     {
         Console.WriteLine("创建欧式的窗户");
     }
}


/// <summary>
/// 欧式的房门
/// </summary>
public class EuropeanDoor : Door
 {
    public override void Create()
    {
        Console.WriteLine("创建欧式的房门");
    }
}
/// <summary>
/// 现代的房顶
/// </summary>
public class ModernizationRoof : Roof
 {
     public override void Create()
    {
         Console.WriteLine("创建现代的房顶");
    }
}

/// <summary>
/// 现代的地板
/// </summary>
public class ModernizationFloor : Floor
{
    public override void Create()
    {
         Console.WriteLine("创建现代的地板");
    }
}
    
/// <summary>
/// 现代的窗户
/// </summary>
public class ModernizationWindow : Window
{
       public override void Create()
      {
           Console.WriteLine("创建现代的窗户");
      }
 }

/// <summary>
/// 现代的房门
/// </summary>
public class ModernizationDoor : Door
{
     public override void Create()
     {
         Console.WriteLine("创建现代的房门");
     }
}
```

## 抽象工厂的实现要点
1. 如果没有应对“多系列对象创建”的需求变化，则没有必要使用`AbstractFactory`模式，这时候使用简单的静态工厂完全可以。
2. "系列对象"指的是这些对象之间有相互依赖、或作用的关系，例如游戏开发场景中“道路”与“房屋”的依赖，“道路”与“地道”的依赖。
3. `AbstractFactory`模式主要在于应对*新系列*的需求变动。其缺点在于难以应对“新对象”的需求变动。
4. `AbstractFactory`模式经常和`FactoryMethod`模式共同组合来应对“对象创建”的需求变化。

## 抽象工厂的优缺点

### 优点
【抽象工厂】模式将系列产品的创建工作延迟到具体工厂的子类中，我们声明工厂类变量的时候是使用的抽象类型，同理，我们使用产品类型也是抽象类型，这样做就尽可能的可以减少客户端代码与具体产品类之间的依赖，从而降低了系统的耦合度。耦合度降低了，对于后期的维护和扩展就更有利，这也就是【抽象工厂】模式的优点所在。
我们通过配置文件来指定具体的工厂类,把依赖关系放到配置文件中。如果有新的需求我们只需要修改配置文件，根本就不需要修改代码了，让客户代码更稳定。

### 缺点

【抽象工厂】模式很难支持增加新产品的变化，这是因为抽象工厂接口中已经确定了可以被创建的产品集合，如果需要添加新产品，此时就必须去修改抽象工厂的接口，这样就涉及到抽象工厂类的以及所有子类的改变，这样也就违背了“开发——封闭”原则。

## 使用场景
如果系统需要多套的代码解决方案，并且每套的代码方案中又有很多相互关联的产品类型，并且在系统中我们可以相互替换的使用一套产品的时候可以使用该模式，客户端不需要依赖具体实现。比较典型的场景是：例如某个产品支持`sql server`、`oracle`、`mysql`数据库时,就可以考虑使用使用抽象工厂模式。典型的案例是.net中的`System.Data.Common.DbProviderFactory`,这个类位于`System.Data.dll`程序集中。该类扮演抽象工厂模式中抽象工厂的角色。
