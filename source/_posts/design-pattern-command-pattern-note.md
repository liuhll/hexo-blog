---
title: 设计模式之命令模式
date: 2018-11-21 22:00:45
categories: "设计模式"
tags:
  - 命令模式
  - 结构型
  - 设计模式
---

# 简介
命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。

# 动机
在软件构建过程中，**行为请求者**与**行为实现者**通常呈现一种“紧耦合”。但在某些场合——比如需要对行为进行“记录、撤销/重做（undo/redo）、事务”等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将“行为请求者”与“行为实现者”解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。

# 意图
将一个请求封装为一个对象，从而使你可用不同的请求对客户（客户程序，也是行为的请求者）进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。　

# 结构图
![command-pattern](command-pattern.png)

# 模式的组成
## 客户角色（Client）
创建具体的命令对象，并且设置命令对象的接收者。注意这个不是我们常规意义上的客户端，而是在组装命令对象和接收者，或许，把这个Client称为装配者会更好理解，因为真正使用命令的客户端是从`Invoker`来触发执行。

## 命令角色（Command）
声明了一个给所有具体命令类实现的抽象接口。

## 具体命令角色（ConcreteCommand）
命令接口实现对象，是“虚”的实现；通常会持有接收者，并调用接收者的功能来完成命令要执行的操作。

## 请求者角色（Invoker）
要求命令对象执行请求，通常会持有命令对象，可以持有很多的命令对象。这个是客户端真正触发命令并要求命令执行相应操作的地方，也就是说相当于使用命令对象的入口。

## 接受者角色（Receiver）
接收者，真正执行命令的对象。任何类都可能成为一个接收者，只要它能够实现命令要求实现的相应功能

# 代码实现
```csharp
namespace 命令模式的实现
{
    /// <summary>
    /// 俗话说：“好吃不如饺子，舒服不如倒着”。今天奶奶发话要吃他大孙子和孙媳妇包的饺子。今天还拿吃饺子这件事来说说命令模式的实现吧。
    /// </summary>
    class Client
    {
        static void Main(string[] args)
        {
            //奶奶想吃猪肉大葱馅的饺子
            PatrickLiuAndWife liuAndLai = new PatrickLiuAndWife();//命令接受者
            Command command = new MakeDumplingsCommand(liuAndLai);//命令
            PaPaInvoker papa = new PaPaInvoker(command); //命令请求者

            //奶奶发布命令
            papa.ExecuteCommand();


            Console.Read();
        }
    }

    //这个类型就是请求者角色--也就是我爸爸的角色，告诉奶奶要吃饺子
    public sealed class PaPaInvoker
    {
        //我爸爸从奶奶那里接受到的命令
        private Command _command;

        //爸爸开始接受具体的命令
        public PaPaInvoker(Command command)
        {
            this._command = command;
        }

        //爸爸给我们下达命令
        public void ExecuteCommand()
        {
            _command.MakeDumplings();
        }
    }

    //该类型就是抽象命令角色--Commmand，定义了命令的抽象接口，任务是包饺子
    public abstract class Command
    {
        //真正任务的接受者
        protected PatrickLiuAndWife _worker;

        protected Command(PatrickLiuAndWife worker)
        {
            _worker = worker;
        }

        //该方法就是抽象命令对象Command的Execute方法
        public abstract void MakeDumplings();
    }

    //该类型是具体命令角色--ConcreteCommand，这个命令完成制作“猪肉大葱馅”的饺子
    public sealed class MakeDumplingsCommand : Command
    {
        public MakeDumplingsCommand(PatrickLiuAndWife worker) : base(worker) { }

        //执行命令--包饺子
        public override void MakeDumplings()
        {
            //执行命令---包饺子
            _worker.Execute("今天包的是农家猪肉和农家大葱馅的饺子");
        }
    }

    //该类型是具体命令接受角色Receiver，具体包饺子的行为是我们夫妻俩来完成的
    public sealed class PatrickLiuAndWife
    {
        //这个方法相当于Receiver类型的Action方法
        public void Execute(string job)
        {
            Console.WriteLine(job);
        }
    }
}
```
# 实现要点
1. `Command`模式的根本目的在于将*行为请求者*与*行为实现者*解耦，在面向对象语言中，常见的实现手段是“将行为抽象为对象”。
2. 实现`Command`接口的具体命令对象`ConcreteCommand`有时候根据需要可能会保存一些额外的状态信息。
3. 通过使用`Composite`组合模式，可以将多个命令封装为一个“复合命令”`MacroCommand`。
4. `Command`模式与C#中的`Delegate`有些类似。但两者定义行为接口的规范有所区别：`Command`以面向对象中的“接口-实现”来定义行为接口规范，更严格，更符合抽象原则；`Delegate`以函数签名来定义行为接口规范，更灵活，但抽象能力比较弱。

5. 使用命令模式会导致某些系统有过多的具体命令类。某些系统可能需要几十个，几百个甚至几千个具体命令类，这会使命令模式在这样的系统里变得不实际。

# 优缺点

## 优点
1. 命令模式使得新的命令很容易被加入到系统里。
2. 可以设计一个命令队列来实现对请求的Undo和Redo操作。
3. 可以较容易地将命令写入日志。
4. 可以把命令对象聚合在一起，合成为合成命令。合成命令式合成模式的应用。

## 缺点
1. 使用命令模式可能会导致系统有过多的具体命令类。这会使得命令模式在这样的系统里变得不实际。

# 使用场景
1. 系统需要支持命令的撤销（`undo`）。命令对象可以把状态存储起来，等到客户端需要撤销命令所产生的效果时，可以调用undo方法把命令所产生的效果撤销掉。命令对象还可以提供redo方法，以供客户端在需要时，再重新实现命令效果。
2. 系统需要在不同的时间指定请求、将请求排队。一个命令对象和原先的请求发出者可以有不同的生命周期。意思为：原来请求的发出者可能已经不存在了，而命令对象本身可能仍是活动的。这时命令的接受者可以在本地，也可以在网络的另一个地址。命令对象可以串行地传送到接受者上去。
3. 如果一个系统要将系统中所有的数据消息更新到日志里，以便在系统崩溃时，可以根据日志里读回所有数据的更新命令，重新调用方法来一条一条地执行这些命令，从而恢复系统在崩溃前所做的数据更新。
4. 系统需要使用命令模式作为`CallBack(回调)`在面向对象系统中的替代。`Callback`即是先将一个方法注册上，然后再以后调用该方法。

# .net的实现
由于.NET有了Delegate，它很少很少用到Command。它只要需要用到行为抽象，它都用`Delegate`去做。