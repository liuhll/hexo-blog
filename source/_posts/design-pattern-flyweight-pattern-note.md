---
title: 设计模式之工厂享元模式
date: 2018-11-15 22:01:08
tags:
---


# 简介
构成事物的最小单元我们称之为“元”，这些单元如果是大量、且重复出现，可以缓存重复出现的单元，达到节省内存的目的，换句说法就是享元是为了节省空间，对于计算机而言就是内存。面向对象很好地解决了系统抽象性的问题（系统抽象性指把系统里面的事物写成类，类可以实例化成为对象，用对象和对象之间的关系来设计系统），在大多数情况下，这样做是不会损及系统的性能的。但是，在某些特殊的应用中，由于对象的数量太大，并且这些大量的对象中有很多是重复的，如果每个对象都单独的创建（C#的语法是new）出来，会给系统带来难以承受的内存开销。比如图形应用中的图元等对象、字处理应用中的字符对象等。

# 动机
在软件系统中，采用纯粹对象方案的问题在于大量细粒度的对象会很快充斥在系统中，从而带来很高的运行时代价——主要指内存需求方面的代价。如何在避免大量细粒度对象问题的同时，让外部客户程序仍然能够透明地使用面向对象的方式来进行操作。

# 意图
运用共享技术有效地支持大量细粒度的对象。　

# 结构图
![享元模式](flyweight-pattern.png)

# 模式的组成
## 抽象享元角色（Flyweight）
此角色是所有的具体享元类的基类，为这些类规定出需要实现的公共接口。那些需要外部状态的操作可以通过调用方法以参数形式传入。

## 具体享元角色（ConcreteFlyweight）
实现抽象享元角色所规定的接口。如果有内部状态的话，可以在类内部定义。

## 享元工厂角色（FlyweightFactory）
本角色负责创建和管理享元角色。本角色必须保证享元对象可以被系统适当地共享，当一个客户端对象调用一个享元对象的时候，享元工厂角色检查系统中是否已经有一个符合要求的享元对象，如果已经存在，享元工厂角色就提供已存在的享元对象，如果系统中没有一个符合的享元对象的话，享元工厂角色就应当创建一个合适的享元对象。

# 代码实现
```csharp
namespace 享元模式的实现
    {
        /// <summary>
        /// 享元模式不是很难，但是有些状态需要单独处理，以下就是该模式的C#实现，有些辅助类，大家应该看得出吧，别混了。
        /// </summary>
        class Client
        {
            static void Main(string[] args)
            {
                //比如，我们现在需要10000个一般士兵，只需这样
                SoldierFactory factory = new SoldierFactory();
                AK47 ak47 = new AK47();
                for (int i = 0; i < 100; i++)
                {
                    Soldier soldier = null;
                    if (i <= 20)
                    {
                        soldier = factory.GetSoldier("士兵" + (i + 1), ak47, SoldierType.Normal);
                    }
                    else
                    {
                        soldier = factory.GetSoldier("士兵" + (i + 1), ak47, SoldierType.Water);
                    }     
                    soldier.Fight();
                }
                //我们有这么多的士兵，但是使用的内存不是很多，因为我们缓存了。
                Console.Read();
            }
        }

        //这些是辅助类型
        public enum SoldierType
        {
            Normal,
            Water
        }

        //该类型就是抽象战士Soldier--该类型相当于抽象享元角色
        public abstract class Soldier
        {
            //通过构造函数初始化士兵的名称
            protected Soldier(string name)
            {
                this.Name = name;
            }

            //士兵的名字
            public string Name { get; private set; }

            //可以传入不同的武器就用不同的活力---该方法相当于抽象Flyweight的Operation方法
            public abstract void Fight();

            public Weapen WeapenInstance { get; set; }
        }

        //一般类型的战士，武器就是步枪---相当于具体的Flyweight角色
        public sealed class NormalSoldier : Soldier
        {
            //通过构造函数初始化士兵的名称
            public NormalSoldier(string name) : base(name) { }

            //执行享元的方法---就是Flyweight类型的Operation方法
            public override void Fight()
            {
                WeapenInstance.Fire("士兵："+Name+" 在陆地执行击毙任务");
            }
        }

        //这是海军陆战队队员，武器精良----相当于具体的Flyweight角色
        public sealed class WaterSoldier : Soldier
        {
            //通过构造函数初始化士兵的名称
            public WaterSoldier(string name) : base(name) { }

            //执行享元的方法---就是Flyweight类型的Operation方法
            public override void Fight()
            {
                WeapenInstance.Fire("士兵："+Name+" 在海中执行击毙任务");
            }
        }

        //此类型和享元没太大关系，可以算是享元对象的状态吧，需要从外部定义
        public abstract class Weapen
        {
            public abstract void Fire(string jobName);
        }

        //此类型和享元没太大关系，可以算是享元对象的状态吧，需要从外部定义
        public sealed class AK47:Weapen
        {
            public override void Fire(string jobName)
            {
                Console.WriteLine(jobName);
            }
        }

        //该类型相当于是享元的工厂---相当于FlyweightFactory类型
        public sealed class SoldierFactory
        {
            private static IList<Soldier> soldiers;

            static SoldierFactory()
            {
                soldiers = new List<Soldier>();
            }

            Soldier mySoldier = null;
            //因为我这里有两种士兵，所以在这里可以增加另外一个参数，士兵类型，原模式里面没有，
            public Soldier GetSoldier(string name, Weapen weapen, SoldierType soldierType)
            {
                foreach (Soldier soldier in soldiers)
                {
                    if (string.Compare(soldier.Name, name, true) == 0)
                    {
                        mySoldier = soldier;
                        return mySoldier;
                    }
                }
                //我们这里就任务名称是唯一的
                if (soldierType == SoldierType.Normal)
                {
                    mySoldier = new NormalSoldier(name);
                }
                else
                {
                    mySoldier = new WaterSoldier(name);
                }
                mySoldier.WeapenInstance = weapen;

                soldiers.Add(mySoldier);
                return mySoldier;
            }
        }
    }
```

# 实现要点
- 面向对象很好地解决了抽象性的问题，但是作为一个运行在机器中的程序实体，我们需要考虑对象的代价问题。Flyweight设计模式主要解决面向对象的代价问题，一般不触及面向对象的抽象性问题。
- Flyweight采用对象共享的做法来降低系统中对象的个数，从而降低细粒度对象给系统带来的内存压力。在具体实现方面，要注意对象状态的处理。
- 对象的数量太大从而导致对象内存开销加大——什么样的数量才算大？这需要我们仔细的根据具体应用情况进行评估，而不能凭空臆断。

# 优缺点

## 优点
1. 享元模式的优点在于它能够极大的减少系统中对象的个数。
2. 享元模式由于使用了外部状态，外部状态相对独立，不会影响到内部状态，所以享元模式使得享元对象能够在不同的环境被共享。

## 缺点
1. 由于享元模式需要区分外部状态和内部状态，使得应用程序在某种程度上来说更加复杂化了。
2. 为了使对象可以共享，享元模式需要将享元对象的状态外部化，而读取外部状态使得运行时间变


# 使用场景
1. 一个系统中有大量的对象；
2. 这些对象耗费大量的内存；
3. 这些对象中的状态大部分都可以被外部化
4. 这些对象可以按照内部状态分成很多的组，当把外部对象从对象中剔除时，每一个组都可以仅用一个对象代替软件系统不依赖这些对象的身份，

满足上面的条件的系统可以使用享元模式。但是使用享元模式需要额外维护一个记录子系统已有的所有享元的表，而这也需要耗费资源，所以，应当在有足够多的享元实例可共享时才值得使用享元模式。
