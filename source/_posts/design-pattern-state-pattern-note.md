---
title: 设计模式之状态模式
date: 2018-11-24 23:08:10
categories: "设计模式"
tags:
- 行为型
- 设计模式
---

# 简介
在状态模式（State Pattern）中，类的行为是基于它的状态改变的。这种类型的设计模式属于行为型模式。
在状态模式中，我们创建表示各种状态的对象和一个行为随着状态对象改变而改变的`context`对象。

# 动机
在软件构建过程中，某些对象的状态如果改变，其行为也会随之而发生变化，比如文档处于只读状态，其支持的行为和读写状态支持的行为就可能完全不同。
如何在运行时根据对象的状态来透明地更改对象的行为？而不会为对象操作和状态转化之间引入紧耦合？

# 意图
允许一个对象在其内部状态改变时改变它的行为。从而使对象看起来似乎修改了其行为

# 结构图
![state-pattern](state-pattern.png)
# 模式的组成
## 环境角色（Context）
也称上下文，定义客户端所感兴趣的接口，并且保留一个具体状态类的实例。这个具体状态类的实例给出此环境对象的现有状态。
## 抽象状态角色（State）
定义一个接口，用以封装环境对象的一个特定的状态所对应的行为。 
## 具体状态角色（ConcreteState）
每一个具体状态类都实现了环境（`Context`）的一个状态所对应的行为

在状态模式结构中需要理解环境类与抽象状态类的作用：
 
环境类实际上就是拥有状态的对象，环境类有时候可以充当`状态管理器`(State Manager)的角色，可以在环境类中对状态进行切换操作。
 
抽象状态类可以是抽象类，也可以是接口，不同状态类就是继承这个父类的不同子类，状态类的产生是由于环境类存在多个状态，同时还满足两个条件：这些状态经常需要切换，在不同的状态下对象的行为不同。因此可以将不同对象下的行为单独提取出来封装在具体的状态类中，使得环境类对象在其内部状态改变时可以改变它的行为，对象看起来似乎修改了它的类，而实际上是由于切换到不同的具体状态类实现的。由于环境类可以设置为任一具体状态类，因此它针对抽象状态类进行编程，在程序运行时可以将任一具体状态类的对象设置到环境类中，从而使得环境类可以改变内部状态，并且改变行为。

# 代码实现
```csharp
namespace 状态模式的实现
    {
        //环境角色---相当于Context类型
        public sealed class Order
        {
            private State current;

            public Order()
            {
                //工作状态初始化为上午工作状态
                current = new WaitForAcceptance();
                IsCancel = false;
            }
            private double minute;
            public double Minute
            {
                get { return minute; }
                set { minute = value; }
            }

            public bool IsCancel { get; set; }

            private bool finish;
            public bool TaskFinished
            {
                get { return finish; }
                set { finish = value; }
            }
            public void SetState(State s)
            {
                current = s;
            }
            public void Action()
            {
                current.Process(this);
            }
        }

        //抽象状态角色---相当于State类型
        public interface State
        {
            //处理订单
            void Process(Order order);
        }

        //等待受理--相当于具体状态角色
        public sealed class WaitForAcceptance : State
        {
            public void Process(Order order)
            {
                System.Console.WriteLine("我们开始受理，准备备货！");
                if (order.Minute < 30 && order.IsCancel)
                {
                    System.Console.WriteLine("接受半个小时之内，可以取消订单！");
                    order.SetState(new CancelOrder());
                    order.Action();
                }
                order.SetState(new AcceptAndDeliver());
                order.TaskFinished = false;
                order.Action();
            }
        }

        //受理发货---相当于具体状态角色
        public sealed class AcceptAndDeliver : State
        {
            public void Process(Order order)
            {
                System.Console.WriteLine("我们货物已经准备好，可以发货了，不可以撤销订单！");
                if (order.Minute < 30 && order.IsCancel)
                {
                    System.Console.WriteLine("接受半个小时之内，可以取消订单！");
                    order.SetState(new CancelOrder());
                    order.Action();
                }
                if (order.TaskFinished==false)
                {
                    order.SetState(new ConfirmationReceipt());
                    order.Action();
                }
            }
        }

        //确认收货---相当于具体状态角色
        public sealed class ConfirmationReceipt : State
        {
            public void Process(Order order)
            {
                System.Console.WriteLine("检查货物，没问题可以就可以签收！");
                order.SetState(new Success());
                order.TaskFinished = false;
                order.Action();
            }
        }

        //交易成功---相当于具体状态角色
        public sealed class Success : State
        {
            public void Process(Order order)
            {
                System.Console.WriteLine("订单结算");
                order.TaskFinished = true;
            }
        }

        //取消订单---相当于具体状态角色
        public sealed class CancelOrder : State
        {
            public void Process(Order order)
            {
                System.Console.WriteLine("检查货物，有问题，取消订单！");
                order.TaskFinished = true;
            }
        }


        public class Client
        {
            public static void Main(String[] args)
            {
                //订单
                Order order = new Order();
                order.Minute = 9;
                order.Action();
                //可以取消订单
                order.IsCancel = true;
                order.Minute = 20;
                order.Action();
                order.Minute = 33;
                order.Action();
                order.Minute = 43;
                order.Action();

                Console.Read();
            }
        }
    }
```

# 实现要点
- `State`模式将所有与一个特定状态相关的行为都放入一个`State`的子类对象中，在对象状态切换时，切换相应的对象；但同时维持`State`的接口，这样实现了具体操作与状态转换之间的解耦。

- 为不同的状态引入不同的对象使得状态转换变得更加明确，而且可以保证不会出现状态不一致的情况，因为转换是原子性的——即要么彻底转换过来，要么不转换。

- 如果`State`对象没有实例变量，那么各个上下文可以共享同一个`State`对象，从而节省对象开销。

# 优缺点

## 优点
1. 封装了转换规则。
2. 枚举可能的状态，在枚举状态之前需要确定状态种类。
3. 将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。
4. 允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块。
5. 可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数。

## 缺点
1. 状态模式的使用必然会增加系统类和对象的个数。
2. 状态模式的结构与实现都较为复杂，如果使用不当将导致程序结构和代码的混乱。
3. 状态模式对“开闭原则”的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源代码，否则无法切换到新增状态；而且修改某个状态类的行为也需修改对应类的源代码。

# 使用场景

1. 对象的行为依赖于它的状态（属性）并且可以根据它的状态改变而改变它的相关行为。
2. 代码中包含大量与对象状态有关的条件语句，这些条件语句的出现，会导致代码的可维护性和灵活性变差，不能方便地增加和删除状态，使客户类与类库之间的耦合增强。在这些条件语句中包含了对象的行为，而且这些条件对应于对象的各种状态