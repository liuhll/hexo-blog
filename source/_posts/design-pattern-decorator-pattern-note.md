---
title: 设计模式之装饰模式
date: 2018-10-28 15:18:43
categories: "设计模式"
tags:
  - 装饰模式
  - 结构型
---

## 介绍
装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。
这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

## 动机
在软件系统中，要给某个类型或者对象增加功能，如果使用“继承”的方案来写代码，就会出现子类暴涨的情况。比如：`IMarbleStyle`是大理石风格的一个功能，`IKeepWarm`是保温的一个接口定义，`IHouseSecurity`是房子安全的一个接口，就三个接口来说，House是我们房子，我们的房子要什么功能就实现什么接口，如果房子要的是复合功能，接口不同的组合就有不同的结果，这样就导致我们子类膨胀严重，如果需要在增加功能，子类会成指数增长。

这个问题的根源在于我们“过度地使用了继承来扩展对象的功能”，由于继承为类型引入的静态特质（所谓静态特质，就是说如果想要某种功能，我们必须在编译的时候就要定义这个类，这也是强类型语言的特点。静态，就是指在编译的时候要确定的东西；动态，是指运行时确定的东西），使得这种扩展方式缺乏灵活性；并且随着子类的增多（扩展功能的增多），各种子类的组合（扩展功能的组合）会导致更多子类的膨胀（多继承）。

## 意图
动态地给一个对象增加一些额外的职责。就增加功能而言，`Decorator模式`比生成子类更为灵活。

## 结构图
![装饰模式](decorator-pattern.png)

## 角色
1. **抽象构件角色（Component）**：给出一个抽象接口，以规范准备接收附加责任的对象。
2. **具体构件角色（Concrete Component）**：定义一个将要接收附加责任的类。
3. **装饰角色（Decorator）**：持有一个构件（Component）对象的实例，并实现一个与抽象构件接口一致的接口。
4. **具体装饰角色（Concrete Decorator）**：负责给构件对象添加上附加的责任。

## 实现
```csharp
namespace 装饰模式的实现
{
    /********************************************/
    // 抽象构件
    /********************************************/
    /// <summary>
    /// 该抽象类就是房子抽象接口的定义，该类型就相当于是Component类型，是饺子馅，需要装饰的，需要包装的
    /// </summary>
    public abstract class House
    {
        //房子的装修方法--该操作相当于Component类型的Operation方法
        public abstract void Renovation();
    }

    /********************************************/
    // 装饰角色
    /********************************************/
    /// <summary>
    /// 该抽象类就是装饰接口的定义，该类型就相当于是Decorator类型，如果需要具体的功能，可以子类化该类型
    /// </summary>
    public abstract class DecorationStrategy:House //关键点之二，体现关系为Is-a，有这这个关系，装饰的类也可以继续装饰了
    {
       //通过组合方式引用Decorator类型，该类型实施具体功能的增加
        //这是关键点之一，包含关系，体现为Has-a
        protected House _house;

        //通过构造器注入，初始化平台实现
        protected DecorationStrategy(House house)
        {
           this._house=house;
        }

       //该方法就相当于Decorator类型的Operation方法
       public override void Renovation()
       {
           if(this._house!=null)
            {
                this._house.Renovation();
            }
        }
    }
 
    /********************************************/
    // 具体构件
    /********************************************/
    /// <summary>
    /// PatrickLiu的房子，我要按我的要求做房子，相当于ConcreteComponent类型，这就是我们具体的饺子馅，我个人比较喜欢韭菜馅
    /// </summary>
    public sealed class PatrickLiuHouse:House
    {
        public override void Renovation()
        {
            Console.WriteLine("装修PatrickLiu的房子");
        }
    }
 
    /********************************************/
    // 具体装饰者
    /********************************************/
   /// <summary>
    /// 具有安全功能的设备，可以提供监视和报警功能，相当于ConcreteDecoratorA类型
    /// </summary>
    public sealed class HouseSecurityDecorator:DecorationStrategy
    {
        public  HouseSecurityDecorator(House house):base(house){}

        public override void Renovation()
        {
            base.Renovation();
            Console.WriteLine("增加安全系统");
        }
    }
 
    /// <summary>
    /// 具有保温接口的材料，提供保温功能，相当于ConcreteDecoratorB类型
    /// </summary>
    public sealed class KeepWarmDecorator:DecorationStrategy
    {
        public  KeepWarmDecorator(House house):base(house){}

        public override void Renovation()
        {
            base.Renovation();
            Console.WriteLine("增加保温的功能");
        }
    }

    /********************************************/
    // 客户端调用
    /********************************************/
   public class Program
   {
      static void Main()
      {
         //这就是我们需要装饰的房子
         House myselfHouse=new PatrickLiuHouse();

         DecorationStrategy securityHouse=new HouseSecurityDecorator(myselfHouse);
         securityHouse.Renovation();
         //房子就有了安全系统了

         //如果我既要安全系统又要保暖呢，继续装饰就行
         DecorationStrategy securityAndWarmHouse=new HouseSecurityDecorator(securityHouse);
         securityAndWarmHouse.Renovation();
      }
   }
}

```
## 要点
1. 通过采用组合、而非继承的手法，**Decorator模式**实现了在运行时动态地扩展对象功能的能力，而且可以根据需要扩展多个功能。避免了单独使用继承带来的*灵活性差*和*多子类衍生问题*。
2. `Component`类在Decorator模式中充当抽象接口的角色，不应该去实现具体的行为。而且`Decorator`类对于`Component`类应该透明——换言之`Component`类无需知道`Decorator`类，`Decorator`类是从外部来扩展`Component`类的功能。
3. `Decorator`类在接口上表现为`is-a` `Component`的继承关系，即`Decorator`类继承了`Component`类所具有的接口。但在实现上又表现为**has-a Component**的组合关系，即`Decorator`类又使用了另外一个`Component`类。我们可以使用一个或者多个`Decorator`对象来“装饰”一个`Component`对象，且装饰后的对象仍然是一个`Component`对象。
4. **Decorator模式**并非解决“多子类衍生的多继承”问题，Decorator模式应用的要点在于解决*主体类在多个方向上的扩展功能*——是为“装饰”的含义。

## 优缺点

### 优点
1. 把抽象接口与其实现解耦。
2. 抽象和实现可以独立扩展，不会影响到对方。
3. 实现细节对客户透明，对用于隐藏了具体实现细节。

### 缺点
- 增加了系统的复杂度

## 使用场景
1. 如果一个系统需要在构件的抽象化角色和具体化角色之间添加更多的灵活性，避免在两个层次之间建立静态的联系。
2. 设计要求实现化角色的任何改变不应当影响客户端，或者实现化角色的改变对客户端是完全透明的。
3. 需要跨越多个平台的图形和窗口系统上。
4. 一个类存在两个独立变化的维度，且两个维度都需要进行扩展。


## .net中的实现
在Net框架中，有一个类型很明显的使用了“装饰模式”，这个类型就是`Stream`。`Stream`类型是一个抽象接口，它在`System.IO`命名空间里面，它其实就是`Component`。`FileStream`、`NetworkStream`、`MemoryStream`都是实体类`ConcreteComponent`。右边的`BufferedStream`、`CryptoStream`是装饰对象，它们都是继承了`Stream`接口的。

![stream类图](stream-classdiagram.png)