---
title: 设计模式之访问者模式
date: 2018-11-27 20:45:58
categories: "设计模式"
tags:
- 行为型
- 设计模式
---

# 简介
在访问者模式（**Visitor Pattern**）中，我们使用了一个访问者类，它改变了元素类的执行算法。通过这种方式，元素的执行算法可以随着访问者改变而改变。这种类型的设计模式属于行为型模式。根据模式，元素对象已接受访问者对象，这样访问者对象就可以处理元素对象上的操作。

# 动机
 在软件构建过程中，由于需求的改变，某些类层次结构中常常需要增加新的行为（方法），如果直接在基类中做这样的更改，将会给子类带来很繁重的变更负担，甚至破坏原有设计。如何在不更改类层次结构的前提下，在运行时根据需要透明地为类层次结构上的各个类动态添加新的操作，从而避免上述问题？

# 意图
表示一个作用于某对象结构中的各个元素的操作。它可以在不改变各元素的类的前提下定义作用于这些元素的新的操作。　

# 结构图 
![visitor-pattern](visitor-pattern.png)

# 模式组成
## 抽象访问者角色（Vistor）
声明一个包括多个访问操作，多个操作针对多个具体节点角色（可以说有多少个具体节点角色就有多少访问操作），使得所有具体访问者必须实现的接口。

## 具体访问者角色（ConcreteVistor）
实现抽象访问者角色中所有声明的接口，也可以说是实现对每个具体节点角色的新的操作。

## 抽象节点角色（Element）
声明一个接受操作，接受一个访问者对象作为参数，如果有其他参数，可以在这个“接受操作”里在定义相关的参数。

## 具体节点角色（ConcreteElement）
实现抽象元素所规定的接受操作。

## 结构对象角色（ObjectStructure）
节点的容器，可以包含多个不同类或接口的容器。

# 代码实现

1. 这是访问者模式第一种代码实例

```csharp
namespace Vistor
{
    //抽象图形定义---相当于“抽象节点角色”Element
    public abstract class Shape
    {
        //画图形
        public abstract void Draw();
        //外界注入具体访问者
        public abstract void Accept(ShapeVisitor visitor);
    }

    //抽象访问者 Visitor
    public abstract class ShapeVisitor
    {
        public abstract void Visit(Rectangle shape);

        public abstract void Visit(Circle shape);

        public abstract void Visit(Line shape);

        //这里有一点要说：Visit方法的参数可以写成Shape吗？就是这样 Visit(Shape shape)，当然可以，但是ShapeVisitor子类Visit方法就需要判断当前的Shape是什么类型，是Rectangle类型，是Circle类型，或者是Line类型。
    }

    //具体访问者 ConcreteVisitor
    public sealed class CustomVisitor : ShapeVisitor
    {
        //针对Rectangle对象
        public override void Visit(Rectangle shape)
        {
            Console.WriteLine("针对Rectangle新的操作！");
        }
        //针对Circle对象
        public override void Visit(Circle shape)
        {
            Console.WriteLine("针对Circle新的操作！");
        }
        //针对Line对象
        public override void Visit(Line shape)
        {
            Console.WriteLine("针对Line新的操作！");
        }
    }

    //矩形----相当于“具体节点角色” ConcreteElement
    public sealed class Rectangle : Shape
    {
        public override void Draw()
        {
            Console.WriteLine("矩形我已经画好！");
        }

        public override void Accept(ShapeVisitor visitor)
        {
            visitor.Visit(this);
        }
    }

    //圆形---相当于“具体节点角色”ConcreteElement
    public sealed class Circle : Shape
    {
        public override void Draw()
        {
            Console.WriteLine("圆形我已经画好！");
        }

        public override void Accept(ShapeVisitor visitor)
        {
            visitor.Visit(this);
        }
    }

    //直线---相当于“具体节点角色” ConcreteElement
    public sealed class Line : Shape
    {
        public override void Draw()
        {
            Console.WriteLine("直线我已经画好！");
        }

        public override void Accept(ShapeVisitor visitor)
        {
            visitor.Visit(this);
        }
    }

    //结构对象角色
    internal class AppStructure
    {
        private ShapeVisitor _visitor;

        public AppStructure(ShapeVisitor visitor)
        {
            this._visitor = visitor;
        }

        public void Process(Shape shape)
        {
            shape.Accept(_visitor);
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            //如果想执行新增加的操作
            ShapeVisitor visitor = new CustomVisitor();
            AppStructure app = new AppStructure(visitor);

            Shape shape = new Rectangle();
            shape.Draw();//执行自己的操作
            app.Process(shape);//执行新的操作


            shape = new Circle();
            shape.Draw();//执行自己的操作
            app.Process(shape);//执行新的操作


            shape = new Line();
            shape.Draw();//执行自己的操作
            app.Process(shape);//执行新的操作


            Console.ReadLine();
        }
    }
}
```

2. 这是访问者模式第二种代码实例

```csharp
namespace Visitor
{
    //抽象访问者角色 Visitor
    public abstract class Visitor
    {
        public abstract void PutTelevision(Television tv);

        public abstract void PutComputer(Computer comp);
    }

    //具体访问者角色 ConcreteVisitor
    public sealed class SizeVisitor : Visitor
    {
        public override void PutTelevision(Television tv)
        {
            Console.WriteLine("按商品大小{0}排放", tv.Size);
        }

        public override void PutComputer(Computer comp)
        {
            Console.WriteLine("按商品大小{0}排放", comp.Size);
        }
    }

    //具体访问者角色 ConcreteVisitor
    public sealed class StateVisitor : Visitor
    {
        public override void PutTelevision(Television tv)
        {
            Console.WriteLine("按商品新旧值{0}排放", tv.State);
        }

        public override void PutComputer(Computer comp)
        {
            Console.WriteLine("按商品新旧值{0}排放", comp.State);
        }
    }

    //抽象节点角色 Element
    public abstract class Goods
    {
        public abstract void Operate(Visitor visitor);

        private int nSize;
        public int Size
        {
            get { return nSize; }
            set { nSize = value; }
        }

        private int nState;
        public int State
        {
            get { return nState; }
            set { nState = value; }
        }
    }

    //具体节点角色 ConcreteElement
    public sealed class Television : Goods
    {
        public override void Operate(Visitor visitor)
        {
            visitor.PutTelevision(this);
        }
    }

    //具体节点角色 ConcreteElement
    public sealed class Computer : Goods
    {
        public override void Operate(Visitor visitor)
        {
            visitor.PutComputer(this);
        }
    }

    //结构对象角色
    public sealed class StoragePlatform
    {
        private IList<Goods> list = new List<Goods>();

        public void Attach(Goods element)
        {
            list.Add(element);
        }

        public void Detach(Goods element)
        {
            list.Remove(element);
        }

        public void Operate(Visitor visitor)
        {
            foreach (Goods g in list)
            {
                g.Operate(visitor);
            }
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            StoragePlatform platform = new StoragePlatform();
            platform.Attach(new Television());
            platform.Attach(new Computer());

            SizeVisitor sizeVisitor = new SizeVisitor();
            StateVisitor stateVisitor = new StateVisitor();

            platform.Operate(sizeVisitor);
            platform.Operate(stateVisitor);

            Console.Read();
        }
    }
}
```
# 实现要点
`Visitor模式`通过所谓双重分发（**double dispatch**）来实现在不更改`Element类`层次结构的前提下，在运行时透明地为类层次结构上的各个类动态添加新的操作。所谓双重分发即`Visitor模式`中间包括了两个多态分发（注意其中的多态机制）：第一个为`accept方法`的多态辨析；第二个为`visit方法`的多态辨析。

设计模式其实是一种*堵漏洞的方式*，但是没有一种设计模式能够堵完所有的漏洞，即使是组合各种设计模式也是一样。每个设计模式都有漏洞，都有它们解决不了的情况或者变化。每一种设计模式都假定了某种变化，也假定了某种不变化。Visitor模式假定的就是操作变化，而Element类层次结构稳定。

# 优缺点

## 优点
1. 访问者模式使得添加新的操作变得容易。如果一些操作依赖于一个复杂的结构对象的话，那么一般而言，添加新的操作会变得很复杂。而使用访问者模式，增加新的操作就意味着添加一个新的访问者类。因此，使得添加新的操作变得容易。
2. 访问者模式使得有关的行为操作集中到一个访问者对象中，而不是分散到一个个的元素类中。这点类似与”中介者模式”。
3. 访问者模式可以访问属于不同的等级结构的成员对象，而迭代只能访问属于同一个等级结构的成员对象。

## 缺点
1. 增加新的元素类变得困难。每增加一个新的元素意味着要在抽象访问者角色中增加一个新的抽象操作，并在每一个具体访问者类中添加相应的具体操作。具体来说，`Visitor模式`的最大缺点在于扩展类层次结构（增添新的`Element子类`），会导致`Visitor类`的改变。因此`Visitor模式`适用于*Element类层次结构稳定，而其中的操作却经常面临频繁改动*。

# 使用场景
1. 如果系统有比较稳定的数据结构，而又有易于变化的算法时，此时可以考虑使用访问者模式。因为访问者模式使得算法操作的添加比较容易。
2. 如果一组类中，存在着相似的操作，为了避免出现大量重复的代码，可以考虑把重复的操作封装到访问者中。（当然也可以考虑使用抽象类了）
3. 如果一个对象存在着一些与本身对象不相干，或关系比较弱的操作时，为了避免操作污染这个对象，则可以考虑把这些操作封装到访问者对象中。