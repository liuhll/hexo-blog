---
title: 设计模式之工厂方法模式
date: 2018-10-14 14:42:15
categories: "设计模式"
tags:
  - 工厂方法模式
  - 创建型
  - 设计模式
---

## 简介
工厂方法模式之把具体产品的创建推迟到子类中，此时工厂类不再负责所有产品的创建，而只是给出具体工厂必须实现的接口，这样工厂方法模式就可以允许系统不修改工厂类逻辑的情况下来添加新产品，这样也就克服了简单工厂模式中缺点。

### 动机
在软件系统的构建过程中，经常面临着*某个对象*的创建工作：由于需求的变化，这个对象（的具体实现）经常面临着剧烈的变化，但是它却拥有比较稳定的接口。
如何应对这种变化？如何提供一种“封装机制”来隔离出“这个易变对象”的变化，从而保持系统中“其他依赖对象的对象”不随着需求改变而改变？

### 意图
定义一个用于创建对象的接口，让子类决定实例化哪一个类。`Factory Method`使得一个类的实例化延迟到子类。 
### 结构图
![工厂模式结构图](factory-method.png)

## 模式组成
### 抽象工厂角色
`Creator`:充当抽象工厂角色，定义工厂类所具有的基本的操作，任何具体工厂都必须继承该抽象类。
### 具体工厂角色
`ConcreteCreator`:充当具体工厂角色，该类必须继承抽象工厂角色，实现抽象工厂定义的方法，用来创建具体产品。
### 抽象产品角色
`Product`:充当抽象产品角色，定义了产品类型所有具有的基本操作，具体产品必须继承该抽象类。
### 具体产品角色
`ConcreteProduct`：充当具体产品角色，实现抽象产品类对定义的抽象方法，由具体工厂类创建，它们之间有一一对应的关系。
## 实现
面向对象的重要原则原则就是: 1. **哪里有变化，就封装哪里**;2. **面向抽象编程**;3. **多组合，少继承**。
例子:
```csharp
/************************************/
/***************产品类***************/
/***********************************/
/// <summary>
/// 汽车抽象类
/// </summary>
public abstract class Car
{
    // 开始行驶
    public abstract void Go();
}
/// <summary>
/// 红旗汽车
/// </summary>
public class HongQiCar : Car
{
    public override void Go()
    {
        Console.WriteLine("红旗汽车开始行驶了！");
    }
}
/// <summary>
/// 奥迪汽车
/// </summary>
public class AoDiCar : Car
{
    public override void Go()
    {
        Console.WriteLine("奥迪汽车开始行驶了");
    }
}

/************************************/
/***************工厂类***************/
/***********************************/
/// <summary>
/// 抽象工厂类
/// </summary>
public abstract class Factory
{
    // 工厂方法
    public abstract Car CreateCar();
}

/// <summary>
/// 红旗汽车工厂类
/// </summary>
public class HongQiCarFactory:Factory
{
    /// <summary>
    /// 负责生产红旗汽车
    /// </summary>
    /// <returns></returns>
    public override Car CreateCar()
    {
        return new HongQiCar();
    }
}    /// <summary>
/// 奥迪汽车工厂类
/// </summary>
public class AoDiCarFactory:Factory
{
    /// <summary>
    /// 负责创建奥迪汽车
    /// </summary>
    /// <returns></returns>
    public override Car CreateCar()
    {
        return new AoDiCar();
    }
}

/**************************/
/***********调用**********/
/************************/
/// <summary>
/// 客户端调用
/// </summary>
class Client
{
    static void Main(string[] args)
    {
        // 初始化创建汽车的两个工厂
        Factory hongQiCarFactory = new HongQiCarFactory();
        Factory aoDiCarFactory = new AoDiCarFactory();
        // 生产一辆红旗汽车
        Car hongQi = hongQiCarFactory.CreateCar();
        hongQi.Go();
        //生产一辆奥迪汽车
        Car aoDi = aoDiCarFactory.CreateCar();
        aoDi.Go();
        Console.Read();
    }
}  
```

## 要点
`Factory Method`模式主要用于隔离类对象的使用者和具体类型之间的耦合关系。面对一个经常变化的具体类型，紧耦合关系会导致软件的脆弱。
`Factory Method`模式通过面向对象的手法，将所要创建的具体对象工作延迟到子类，从而实现一种扩展（而非更改）的策略，较好地解决了这种紧耦合关系。

## 构建者对象的比较
1. `Factory Method`模式解决**单个对象**的需求变化；
2. `AbstractFactory`模式解决**系列对象**的需求变化；
3. `Builder`模式解决**对象部分**的需求变化；

## 优缺点

### 优点
1. 在工厂方法中，用户只需要知道所要产品的具体工厂，无须关系具体的创建过程，甚至不需要具体产品类的类名。
2. 在系统增加新的产品时，我们只需要添加一个具体产品类和对应的实现工厂，无需对原工厂进行任何修改，很好地符合了`开闭原则`。
### 缺点
- 每次增加一个产品时，都需要增加一个具体类和对象实现工厂，是的系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。这并不是什么好事。

## 工厂方法模式使用的场景
1. 一个类不知道它所需要的对象的类。在工厂方法模式中，我们不需要具体产品的类名，我们只需要知道创建它的具体工厂即可。
2. 一个类通过其子类来指定创建那个对象。在工厂方法模式中，对于抽象工厂类只需要提供一个创建产品的接口，而由其子类来确定具体要创建的对象，在程序运行时，子类对象将覆盖父类对象，从而使得系统更容易扩展。
3. 将创建对象的任务委托给多个工厂子类中的某一个，客户端在使用时可以无须关心是哪一个工厂子类创建产品子类，需要时再动态指定。