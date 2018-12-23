---
title: 设计模式之中介者模式
date: 2018-11-22 21:04:25
categories: "设计模式"
tags:
  - 行为型
  - 设计模式
---

# 简介
中介者模式（Mediator Pattern）是用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。中介者模式属于行为型模式。

# 动机
在软件构建过程中，经常会出现多个对象互相关联交互的情况，对象之间常常会维持一种复杂的引用关系，如果遇到一些需求的更改，这种直接的引用关系将面临不断地变化。
在这种情况下，我们可使用一个“中介对象”来管理对象间的关联关系，避免相互交互的对象之间的紧耦合引用关系，从而更好地抵御变化。

# 意图
定义了一个中介对象来封装一系列对象之间的交互关系。中介者使各个对象之间不需要显式地相互引用，从而使耦合性降低，而且可以独立地改变它们之间的交互行为。

# 结构图
![mediator-pattern](mediator-pattern.png)

# 角色组成
## 抽象中介者角色（Mediator）
在里面定义各个同事之间交互需要的方法，可以是公共的通信方法，也可以是小范围的交互方法。

## 具体中介者角色（ConcreteMediator）
它需要了解并维护各个同事对象，并负责具体的协调各同事对象的交互关系。

## 抽象同事类（Colleague）
通常为抽象类，主要约束同事对象的类型，并实现一些具体同事类之间的公共功能，比如，每个具体同事类都应该知道中介者对象，也就是具体同事类都会持有中介者对象，都可以到这个类里面。

## 具体同事类（ConcreteColleague）
实现自己的业务，需要与其他同事通信时候，就与持有的中介者通信，中介者会负责与其他同事类交互。

# 代码实现
```csharp
namespace 中介者模式的实现
{
    //抽象中介者角色
    public interface Mediator
    {
        void Command(Department department);
    }

    //总经理--相当于具体中介者角色
    public sealed class President : Mediator
    {
        //总经理有各个部门的管理权限
        private Financial _financial;
        private Market _market;
        private Development _development;

        public void SetFinancial(Financial financial)
        {
            this._financial = financial;
        }
        public void SetDevelopment(Development development)
        {
            this._development = development;
        }
        public void SetMarket(Market market)
        {
            this._market = market;
        }

        public void Command(Department department)
        {
            if (department.GetType() == typeof(Market))
            {
                _financial.Process();
            }
        }
    }

    //同事类的接口
    public abstract class Department
    {
        //持有中介者(总经理)的引用
        private Mediator mediator;

        protected Department(Mediator mediator)
        {
            this.mediator = mediator;
        }

        public Mediator GetMediator
        {
            get { return mediator; }
            private set { this.mediator = value; }
        }

        //做本部门的事情
        public abstract void Process();

        //向总经理发出申请
        public abstract void Apply();
    }

    //开发部门
    public sealed class Development : Department
    {
        public Development(Mediator m) : base(m) { }

        public override void Process()
        {
            Console.WriteLine("我们是开发部门，要进行项目开发，没钱了，需要资金支持！");
        }

        public override void Apply()
        {
            Console.WriteLine("专心科研，开发项目！");
        }
    }

    //财务部门
    public sealed class Financial : Department
    {
        public Financial(Mediator m) : base(m) { }

        public override void Process()
        {
            Console.WriteLine("汇报工作！没钱了，钱太多了！怎么花?");
        }

        public override void Apply()
        {
            Console.WriteLine("数钱！");
        }
    }

    //市场部门
    public sealed class Market : Department
    {
        public Market(Mediator mediator) : base(mediator) { }

        public override void Process()
        {
            Console.WriteLine("汇报工作！项目承接的进度，需要资金支持！");
            GetMediator.Command(this);
        }

        public override void Apply()
        {
            Console.WriteLine("跑去接项目！");
        }
    }


    class Program
    {
        static void Main(String[] args)
        {
            // 中介者
            President mediator = new President();

            // 同事类
            Market market = new Market(mediator);
            Development development = new Development(mediator);
            Financial financial = new Financial(mediator);

            mediator.SetFinancial(financial);
            mediator.SetDevelopment(development);
            mediator.SetMarket(market);

            market.Process();
            market.Apply();

            Console.Read();
        }
    }
}
```
# 实现要点
将多个对象间复杂的关联关系解耦，`Mediator模式`将多个对象间的控制逻辑进行集中管理，变“多个对象互相关联”为“多个对象和一个中介者关联”，简化了系统的维护，抵御了可能的变化。随着控制逻辑的复杂化，`Mediator`具体对象的实现可能相当复杂。这时候可以对`Mediator`对象进行分解处理。

## 门面模式与中介者模式的区别
- `Facade模式`是解耦系统外到系统内（单向）的对相关联关系
- `Mediator模式`是解耦系统内各个对象之间（双向）的关联关系

# 优缺点 

## 优点 
1. 松散耦合
  - 中介者模式通过把多个同事对象之间的交互封装到中介对象里面，从而使得对象之间松散耦合，基本上可以做到互不依赖。这样一来，同时对象就可以独立的变化和复用，不再“牵一发动全身”

2. 集中控制交互
  - 多个同事对象的交互，被封装在中介者对象里面集中管理，使得这些交互行为发生变化的时候，只需要修改中介者就可以了。

3. 多对多变为一对多
   - 没有中介者模式的时候，同事对象之间的关系通常是多对多，引入中介者对象后，中介者和同事对象的关系通常变为双向的一对多，这会让对象的关系更容易理解和实现。

## 缺点
1. 过多集中化
  -  如果同事对象之间的交互非常多，而且比较复杂，当这些复杂性全都集中到中介者的时候，会导致中介者对象变的十分复杂，而且难于维护和管理
  
# 总结
如果不使用中介者模式的话，各个同事对象将会相互进行引用，如果每个对象都与多个对象进行交互时，将会形成如下图所示的网状结构:
![不使用中介者模式](mediator-pattern1.jpg)
如果不使用中介者模式的话，每个对象之间过度耦合，这样的既不利于类的复用也不利于扩展。如果引入了中介者模式，那么对象之间的关系将变成星型结构，采用中介者模式之后会形成如下图所示的结构：
![使用中介者模式](mediator-pattern2.jpg)

使用中介者模式之后，任何一个类的变化，只会影响中介者和类本身，不像之前的设计，任何一个类的变化都会引起其关联所有类的变化。这样的设计大大减少了系统的耦合度
