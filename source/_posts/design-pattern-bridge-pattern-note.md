---
title: 设计模式之桥接模式
date: 2018-10-24 22:25:48
categories: "设计模式"
tags:
  - 结构型
  - 设计模式
---

## 简介
【桥接模式】，也有叫【桥模式】的，英文名称：Bridge Pattern。可以通过名称看出这个模式是根据名称猜肯定是连接什么东西的。因为桥在我们现实生活中经常是连接着A地和B地，再往后来发展，桥引申为一种纽带。
## 动机 
桥接（`Bridge`）是用于把抽象化与实现化解耦，使得二者可以独立变化。这种类型的设计模式属于结构型模式，它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。
这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。这两种类型的类可被结构化改变而互不影响。桥是针对桥的使用环境来说的，解决了跨越和衔接的问题。

## 意图
将抽象部分与实现部分分离，使它们都可以独立的变化。

## 结构图
![桥接模式](bridge-pattern.png)

## 模式的组成
1. **抽象化角色(Abstraction)：** 抽象化给出的定义，并保存一个对实现化对象（Implementor）的引用。
2. **修正抽象化角色(Refined Abstraction)：** 扩展抽象化角色，改变和修正父类对抽象化的定义。
3. **实现化角色(Implementor)：** 这个角色给出实现化角色的接口，但不给出具体的实现。必须指出的是，这个接口不一定和抽象化角色的接口定义相同，实际上，这两个接口可以非常不一样。实现化角色应当只给出底层操作，而抽象化角色应当只给出基于底层操作的更高一层的操作。
4. **具体实现化角色(Concrete Implementor)：** 这个角色给出实现化角色接口的具体实现。

## 实现
案例: 以数据库为例来写该模式的实现。每种数据库都有自己的版本，但是每种数据库在不同的平台上实现又是不一样的。比如：微软的SqlServer数据库，该数据库它有2000版本、2005版本、2006版本、2008版本，后面还会有更新的版本。并且这些版本都是运行在Windows操作系统下的，如果要提供Lunix操作系统下的SqlServer怎么办呢？如果又要提供IOS操作系统下的SqlServer数据库该怎么办呢？这个情况就可以使用桥接模式，也就是Brige模式。

```csharp
namespace 桥接模式的实现
{

//////////////////////////////////////////////////////////////
// 抽象化角色
//////////////////////////////////////////////////////////////
/// <summary>
/// 该抽象类就是抽象接口的定义，该类型就相当于是Abstraction类型
/// </summary>
public abstract class Database
{
    //通过组合方式引用平台接口，此处就是桥梁，该类型相当于Implementor类型
    protected PlatformImplementor _implementor;
    //通过构造器注入，初始化平台实现
    protected Database(PlatformImplementor implementor)
    {
        this._implementor = implementor;
    }
    //创建数据库--该操作相当于Abstraction类型的Operation方法
    public abstract void Create();
}


//////////////////////////////////////////////////////////////
// 实现化角色
//////////////////////////////////////////////////////////////

/// <summary>
/// 该抽象类就是实现接口的定义，该类型就相当于是Implementor类型
/// </summary>
public abstract class PlatformImplementor
{
    //该方法就相当于Implementor类型的OperationImpl方法
    public abstract void Process();
}


//////////////////////////////////////////////////////////////
// 修正抽象化角色
//////////////////////////////////////////////////////////////
/// <summary>
/// SqlServer2000版本的数据库，相当于RefinedAbstraction类型
/// </summary>
public class SqlServer2000 : Database
{
    //构造函数初始化
    public SqlServer2000(PlatformImplementor implementor) : base(implementor) { }
    public override void Create()
    {
        this._implementor.Process();
    }
}

/// <summary>
/// SqlServer2005版本的数据库，相当于RefinedAbstraction类型
/// </summary>
public class SqlServer2005 : Database
{
    //构造函数初始化
    public SqlServer2005(PlatformImplementor implementor) : base(implementor) { }
    public override void Create()
    {
        this._implementor.Process();
    }
}


//////////////////////////////////////////////////////////////
// 具体实现化角色
//////////////////////////////////////////////////////////////

/// <summary>
/// SqlServer2000版本的数据库针对Unix操作系统具体的实现，相当于ConcreteImplementorA类型
/// </summary>
public class SqlServer2000UnixImplementor : PlatformImplementor
{
    public override void Process()
    {
        Console.WriteLine("SqlServer2000针对Unix的具体实现");
    }
}

/// <summary>
/// SqlServer2005版本的数据库针对Unix操作系统的具体实现，相当于ConcreteImplementorB类型
/// </summary>
public sealed class SqlServer2005UnixImplementor : PlatformImplementor
{
    public override void Process()
    {
        Console.WriteLine("SqlServer2005针对Unix的具体实现");
    }
}

//////////////////////////////////////////////////////////////
// 客户端
//////////////////////////////////////////////////////////////

  public class Program
  {
      static void Main()
      {
          PlatformImplementor SqlServer2000UnixImp = new SqlServer2000UnixImplementor();
          //还可以针对不同平台进行扩展，也就是子类化，这个是独立变化的
          Database SqlServer2000Unix = new SqlServer2000(SqlServer2000UnixImp);
          //数据库版本也可以进行扩展和升级，也进行独立的变化。
          //以上就是两个维度的变化。
       //就可以针对Unix执行操作了
       SqlServer2000Unix.Create();
      }
  }
}
```

## 要点
1. `Bridge模式`使用“对象间的组合关系”解耦了抽象和实现之间固有的绑定关系，使得抽象和实现可以沿着各自的维度来变化。
2. 所谓抽象和实现沿着各自维度的变化，即*子类化*它们，得到各个子类之后，便可以任意组合它们，从而获得不同平台上的不同型号。
3. `Bridge模式`有时候类似于多继承方案，但是多继承方案往往违背了类的单一职责原则（即一个类只有一个变化的原因），复用性比较差。`Bridge模式`是比多继承方案更好的解决方法。
4. `Bridge模式`的应用一般在*两个非常强的变化维度*，有时候即使有两个变化的维度，但是某个方向的变化维度并不剧烈——换言之两个变化不会导致纵横交错的结果，并不一定要使用Bridge模式。

## 优缺点

### 优点
1. 把抽象接口与其实现解耦。
2. 抽象和实现可以独立扩展，不会影响到对方。
3. 实现细节对客户透明，对用于隐藏了具体实现细节。

### 缺点
增加了系统的复杂度

## 使用场景

1. 如果一个系统需要在构件的抽象化角色和具体化角色之间添加更多的灵活性，避免在两个层次之间建立静态的联系。
2. 设计要求实现化角色的任何改变不应当影响客户端，或者实现化角色的改变对客户端是完全透明的。
3. 需要跨越多个平台的图形和窗口系统上。
4.  一个类存在两个独立变化的维度，且两个维度都需要进行扩展。