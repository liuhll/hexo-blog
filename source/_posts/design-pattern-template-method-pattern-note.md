---
title: 设计模式之模板方法模式
date: 2018-11-19 19:28:36
categories: "设计模式"
tags:
- 模板方法
- 创建性
- 设计模式
---

# 简介
在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。这种类型的设计模式属于行为型模式。

# 动机
在软件构建过程中，“行为请求者”与“行为实现者”通常呈现一种“紧耦合”。但在某些场合——比如需要对行为进行“记录、撤销/重做（undo/redo）、事务”等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将“行为请求者”与“行为实现者”解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。

# 意图
将一个请求封装为一个对象，从而使你可用不同的请求对客户（客户程序，也是行为的请求者）进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。

# 结构图
![模板方法结构图](template-method-pattern.png)

# 模式的组成
## 抽象类角色（AbstractClass）
定义一个模板方法（TemplateMethod），在该方法中包含着一个算法的骨架，具体的算法步骤是PrimitiveOperation1方法和PrimitiveOperation2方法，该抽象类的子类将重定义`PrimitiveOperation1`和`PrimitiveOperation2`操作。

## 具体类角色（ConcreteClass）
实现`PrimitiveOperation1`方法和`PrimitiveOperation2`方法以完成算法中与特定子类（Client）相关的内容。

> **Notes**
> 在模板方法模式中，`AbstractClass`中的`TemplateMethod`提供了一个标准模板，该模板包含`PrimitiveOperation1`和`PrimitiveOperation2`两个方法，这两个方法的内容Client可以根据自己的需要重写。

# 代码实现
```csharp
namespace 模板方法模式的实现
{
    /// <summary>
    /// 好吃不如饺子，舒服不如倒着，我最喜欢吃我爸爸包的饺子，今天就拿吃饺子这件事来看看模板方法的实现吧
    /// </summary>
    class Client
    {
        static void Main(string[] args)
        {
            //现在想吃绿色面的，猪肉大葱馅的饺子
            AbstractClass fan = new ConcreteClass();
            fan.EatDumplings();

            Console.WriteLine();
            //过了段时间，我开始想吃橙色面的，韭菜鸡蛋馅的饺子
            fan = new ConcreteClass2();
            fan.EatDumplings();


            Console.Read();
        }
    }


    //该类型就是抽象类角色--AbstractClass，定义做饺子的算法骨架，这里有三步骤，当然也可以有多个步骤，根据实际需要而定
    public abstract class AbstractClass
    {
        //该方法就是模板方法，方法里面包含了做饺子的算法步骤，模板方法可以返回结果，也可以是void类型，视具体情况而定
        public void EatDumplings()
        {
            //和面
            MakingDough();
            //包馅
            MakeDumplings();
            //煮饺子
            BoiledDumplings();

            Console.WriteLine("饺子真好吃！");
        }

        //要想吃饺子第一步肯定是“和面”---该方法相当于算法中的某一步
        public abstract void MakingDough();

        //要想吃饺子第二部是“包饺子”---该方法相当于算法中的某一步
        public abstract void MakeDumplings();

        //要想吃饺子第三部是“煮饺子”---该方法相当于算法中的某一步
        public abstract void BoiledDumplings();
    }

    //该类型是具体类角色--ConcreteClass，我想吃绿色面皮，猪肉大葱馅的饺子
    public sealed class ConcreteClass : AbstractClass
    {
        //要想吃饺子第一步肯定是“和面”---该方法相当于算法中的某一步
        public override void MakingDough()
        {
            //我想要面是绿色的，绿色健康嘛，就可以在此步定制了
            Console.WriteLine("在和面的时候加入芹菜汁，和好的面就是绿色的");
        }

        //要想吃饺子第二部是“包饺子”---该方法相当于算法中的某一步
        public override void MakeDumplings()
        {
            //我想吃猪肉大葱馅的，在此步就可以定制了
            Console.WriteLine("农家猪肉和农家大葱，制作成馅");
        }

        //要想吃饺子第三部是“煮饺子”---该方法相当于算法中的某一步
        public override void BoiledDumplings()
        {
            //我想吃大铁锅煮的饺子，有家的味道，在此步就可以定制了
            Console.WriteLine("用我家的大铁锅和大木材煮饺子");
        }
    }

    //该类型是具体类角色--ConcreteClass2，我想吃橙色面皮，韭菜鸡蛋馅的饺子
    public sealed class ConcreteClass2 : AbstractClass
    {
        //要想吃饺子第一步肯定是“和面”---该方法相当于算法中的某一步
        public override void MakingDough()
        {
            //我想要面是橙色的，加入胡萝卜汁就可以。在此步定制就可以了。
            Console.WriteLine("在和面的时候加入胡萝卜汁，和好的面就是橙色的");
        }

        //要想吃饺子第二部是“包饺子”---该方法相当于算法中的某一步
        public override void MakeDumplings()
        {
            //我想吃韭菜鸡蛋馅的，在此步就可以定制了
            Console.WriteLine("农家鸡蛋和农家韭菜，制作成馅");
        }

        //要想吃饺子第三部是“煮饺子”---该方法相当于算法中的某一步
        public override void BoiledDumplings()
        {
            //此处没要求
            Console.WriteLine("可以用一般煤气和不粘锅煮就可以");
        }
    }
}
```
# 实现要点
1. TemplateMethod模式是一种非常基础性的设计模式，在面向对象系统中大量应用。它用最简洁的机制（基础、多态）为很多应用程序框架提供了灵活的扩展点，是代码复用方面的基本实现结构。
2. 在具体实现方面，被TemplateMethod调用的虚方法可以具有实现，也可以没有任何实现（抽象方法或虚方法）。但一般推荐将它们设置为protected方法使得只有子类可以访问它们。
3. 模板方法模式通过对子类的扩展增加新的行为，符合“开闭原则”。

# 适用场景
1. 一次性实现一个算法的不变部分，并将可变的行为留给子类来实现。
2. 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复。
3. 控制子类扩展。模板方法只允许在特定点进行扩展，而模板部分则是稳定的。
