---
title: 设计模式之策略模式
date: 2018-11-25 21:08:04
categories: "设计模式"
tags:
- 行为型
- 设计模式
---

# 简介
在策略模式（**Strategy Pattern**）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的`context`对象。策略对象改变`context` 对象的执行算法。

# 动机
在软件构建过程中，某些对象使用的算法可能多种多样，经常改变，如果将这些算法都编码到对象中，将会使对象变得异常复杂；而且有时候支持不使用的算法也是一个性能负担。如何在运行时根据需要透明地更改对象的算法？将算法与对象本身解耦，从而避免上述问题？

# 意图
定义一系列算法，把它们一个个封装起来，并且使它们可互相替换。该模式使得算法可独立于使用它的客户而变化。

# 结构图
![stragety-pattern](stragety-pattern.png)

# 模式的组成
## 环境角色（Context）
- 持有一个`Strategy`类的引用。
- 需要使用`ConcreteStrategy`提供的算法。
- 内部维护一个`Strategy`的实例。
- 负责动态设置运行时`Strategy`具体的实现算法。
- 负责跟`Strategy`之间的交互和数据传递

## 抽象策略角色（Strategy）
- 定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，`Context`使用这个接口调用不同的算法，一般使用接口或抽象类实现。

## 具体策略角色（ConcreteStrategy）
- 实现了`Strategy`定义的接口，提供具体的算法实现

# 代码实现

```csharp
namespace 策略模式的实现
{
    //环境角色---相当于Context类型
    public sealed class SalaryContext
    {
        private ISalaryStrategy _strategy;

        public SalaryContext(ISalaryStrategy strategy)
        {
            this._strategy = strategy;
        }

        public ISalaryStrategy ISalaryStrategy
        {
            get { return _strategy; }
            set { _strategy = value; }
        }

        public void GetSalary(double income)
        {
            _strategy.CalculateSalary(income);
        }
    }

    //抽象策略角色---相当于Strategy类型
    public interface ISalaryStrategy
    {
        //工资计算
        void CalculateSalary(double income);
    }

    //程序员的工资--相当于具体策略角色ConcreteStrategyA
    public sealed class ProgrammerSalary : ISalaryStrategy
    {
        public void CalculateSalary(double income)
        {
            Console.WriteLine("我的工资是：基本工资(" + income + ")底薪(" + 8000 + ")+加班费+项目奖金（10%）");
        }
    }

    //普通员工的工资---相当于具体策略角色ConcreteStrategyB
    public sealed class NormalPeopleSalary : ISalaryStrategy
    {
        public void CalculateSalary(double income)
        {
            Console.WriteLine("我的工资是：基本工资(" + income + ")底薪(3000)+加班费");
        }
    }

    //CEO的工资---相当于具体策略角色ConcreteStrategyC
    public sealed class CEOSalary : ISalaryStrategy
    {
        public void CalculateSalary(double income)
        {
            Console.WriteLine("我的工资是：基本工资(" + income + ")底薪(20000)+项目奖金(20%)+公司股票");
        }
    }


    public class Client
    {
        public static void Main(String[] args)
        {
            //普通员工的工资
            SalaryContext context = new SalaryContext(new NormalPeopleSalary());
            context.GetSalary(3000);

            //CEO的工资
            context.ISalaryStrategy = new CEOSalary();
            context.GetSalary(6000);

            Console.Read();
        }
    }
}
```

# 实现要点
`Strategy`及其子类为组件提供了一系列可重用的算法，从而可以使得类型在运行时方便地根据需要在各个算法之间进行切换，所谓封装算法，支持算法的变化。`Strategy`模式提供了用条件判断语句以外的另一种选择，消除条件判断语句，就是在解耦合。含有许多条件判断语句的代码通常都需要`Strategy`模式。

与`State`类似，如果`Strategy`对象没有实例变量，那么各个上下文可以共享一个`Strategy`对象，从而节省对象开销。`Strategy`模式适用的是算法结构中整个算法的改变，而不是算法中某个部分的改变。

`Template Method`方法：执行算法的步骤协议是本身放在抽象类里面的，允许一个通用的算法操作多个可能实现

`Strategy`模式：执行算法的协议是在具体类，每个具体实现有不同通用算法来做。

# 优缺点

## 优点
1. 策略类之间可以自由切换。由于策略类都实现同一个接口，所以使它们之间可以自由切换。
2. 易于扩展。增加一个新的策略只需要添加一个具体的策略类即可，基本不需要改变原有的代码。
3. 避免使用多重条件选择语句，充分体现面向对象设计思想。

## 缺点
1. 客户端必须知道所有的策略类，并自行决定使用哪一个策略类。这点可以考虑使用IOC容器和依赖注入的方式来解决
        
2. 策略模式会造成很多的策略类。

# 使用场景
1. 一个系统需要动态地在几种算法中选择一种的情况下。那么这些算法可以包装到一个个具体的算法类里面，并为这些具体的算法类提供一个统一的接口。

2. 如果一个对象有很多的行为，如果不使用合适的模式，这些行为就只好使用多重的`if-else`语句来实现，此时，可以使用策略模式，把这些行为转移到相应的具体策略类里面，就可以避免使用难以维护的多重条件选择语句，并体现面向对象涉及的概念。
