---
title: 设计模式之组合模式
date: 2018-10-28 15:18:43
categories: "设计模式"
tags:
  - 设计模式
  - 结构型
---

## 介绍
组合模式（Composite Pattern），又叫*部分整体模式*，是用于把一组相似的对象当作一个单一的对象。组合模式依据树形结构来组合对象，用来表示部分以及整体层次。这种类型的设计模式属于结构型模式，它创建了对象组的树形结构。
这种模式创建了一个包含自己对象组的类。该类提供了修改相同对象组的方式。
在组合模式中,包含的这些东西或者说是对象，可以分为两类，一类是：容器对象，可以包含其他的子对象；另一类是：叶子对象，这类对象是不能在包含其他对象的对象了。

## 动机
客户代码过多地依赖于对象容器（对象容器是对象的容器，细细评味）复杂的内部实现结构，对象容器内部实现结构（而非抽象接口）的变化将引起客户代码的频繁变化，带来了代码的维护性、扩展性等方面的弊端。如何将“客户代码与复杂的对象容器结构”解耦？如何让对象容器自己来实现自身的复杂结构，从而使得客户代码就像处理简单对象一样来处理复杂的对象容器？

## 意图
将对象组合成树形结构以表示“部分-整体”的层次结构。Composite使得用户对单个对象和组合对象的使用具有一致性。

## 结构图
![组合模式](composite-pattern.png)

## 角色
1. **抽象构件角色（Component）**：这是一个抽象角色，它给参加组合的对象定义出了公共的接口及默认行为，可以用来管理所有的子对象（在透明式的组合模式是这样的）。在安全式的组合模式里，构件角色并不定义出管理子对象的方法，这一定义由树枝结构对象给出。
2. **树叶构件角色（Leaf）**：树叶对象是没有下级子对象的对象，定义出参加组合的原始对象的行为。（原始对象的行为可以理解为没有容器对象管理子对象的方法，或者 【原始对象行为】+【管理子对象的行为（Add，Remove等）】=面对客户代码的接口行为集合）
3. **树枝构件角色（Composite）**：代表参加组合的有下级子对象的对象，树枝对象给出所有管理子对象的方法实现，如Add、Remove等。

> **notes**
> 组合模式实现的最关键的地方是——简单对象和复合对象必须实现相同的接口。

## 实现
1. 透明式的组合模式
  - **抽象构件角色** 定义的接口行为集合包含两个部分，一部分是叶子对象本身所包含的行为（比如Operation），另外一部分是容器对象本身所包含的管理子对象的行为(`Add`,`Remove`)。这个抽象构件必须同时包含这两类对象所有的行为，客户端代码才会透明的使用，无论调用容器对象还是叶子对象，接口方法都是一样的，这就是透明

2. 安全式的组合模式
  - 只定义叶子对象的方法，确切的说这个抽象构件只定义两类对象共有的行为，然后容器对象的方法定义在“树枝构件角色”上，这样叶子对象有叶子对象的方法，容器对象有容器对象的方法，这样责任很明确，当然调用肯定不会抛出异常了。

### 透明诗的组合模式
```csharp
namespace 透明式的组合模式的实现
{
    /// <summary>
    /// 该抽象类就是文件夹抽象接口的定义，该类型就相当于是抽象构件Component类型
    /// </summary>
    public abstract class Folder
    {
        //增加文件夹或文件
        public abstract void Add(Folder folder);

        //删除文件夹或者文件
        public abstract void Remove(Folder folder);

        //打开文件或者文件夹--该操作相当于Component类型的Operation方法
        public abstract void Open();
    }

    /// <summary>
    /// 该Word文档类就是叶子构件的定义，该类型就相当于是Leaf类型，不能在包含子对象
    /// </summary>
    public sealed class Word : Folder
    {
        //增加文件夹或文件
        public override void Add(Folder folder)
        {
            throw new Exception("Word文档不具有该功能");
        }

        //删除文件夹或者文件
        public override void Remove(Folder folder)
        {
            throw new Exception("Word文档不具有该功能");
        }

        //打开文件--该操作相当于Component类型的Operation方法
        public override void Open()
        {
            Console.WriteLine("打开Word文档，开始进行编辑");
        }
    }

    /// <summary>
    /// SonFolder类型就是树枝构件，由于我们使用的是“透明式”，所以Add,Remove都是从Folder类型继承下来的
    /// </summary>
    public class SonFolder : Folder
    {
        //增加文件夹或文件
        public override void Add(Folder folder)
        {
            Console.WriteLine("文件或者文件夹已经增加成功");
        }

        //删除文件夹或者文件
        public override void Remove(Folder folder)
        {
            Console.WriteLine("文件或者文件夹已经删除成功");
        }

        //打开文件夹--该操作相当于Component类型的Operation方法
        public override void Open()
        {
            Console.WriteLine("已经打开当前文件夹");
        }
    }

    public class Program
    {
        static void Main()
        {

            Folder myword = new Word();

            myword.Open();//打开文件，处理文件

            myword.Add(new SonFolder());//抛出异常
            myword.Remove(new SonFolder());//抛出异常


            Folder myfolder = new SonFolder();
            myfolder.Open();//打开文件夹

            myfolder.Add(new SonFolder());//成功增加文件或者文件夹
            myfolder.Remove(new SonFolder());//成功删除文件或者文件夹

            Console.Read();
        }
    }
}
```
### 安全式的组合模式
```csharp
namespace 安全式的组合模式的实现
{
    /// <summary>
    /// 该抽象类就是文件夹抽象接口的定义，该类型就相当于是抽象构件Component类型
    /// </summary>
    public abstract class Folder //该类型少了容器对象管理子对象的方法的定义，换了地方，在树枝构件也就是SonFolder类型
    {
        //打开文件或者文件夹--该操作相当于Component类型的Operation方法
        public abstract void Open();
    }

    /// <summary>
    /// 该Word文档类就是叶子构件的定义，该类型就相当于是Leaf类型，不能在包含子对象
    /// </summary>
    public sealed class Word : Folder  //这类型现在很干净
    {
        //打开文件--该操作相当于Component类型的Operation方法
        public override void Open()
        {
            Console.WriteLine("打开Word文档，开始进行编辑");
        }
    }

    /// <summary>
    /// SonFolder类型就是树枝构件，现在由于我们使用的是“安全式”，所以Add,Remove都是从此处开始定义的
    /// </summary>
    public abstract class SonFolder : Folder //这里可以是抽象接口，可以自己根据自己的情况而定
    {
        //增加文件夹或文件
        public abstract void Add(Folder folder);

        //删除文件夹或者文件
        public abstract void Remove(Folder folder);

        //打开文件夹--该操作相当于Component类型的Operation方法
        public override void Open()
        {
            Console.WriteLine("已经打开当前文件夹");
        }
    }

    /// <summary>
    /// NextFolder类型就是树枝构件的实现类
    /// </summary>
    public sealed class NextFolder : SonFolder
    {
        //增加文件夹或文件
        public override void Add(Folder folder)
        {
            Console.WriteLine("文件或者文件夹已经增加成功");
        }

        //删除文件夹或者文件
        public override void Remove(Folder folder)
        {
            Console.WriteLine("文件或者文件夹已经删除成功");
        }

        //打开文件夹--该操作相当于Component类型的Operation方法
        public override void Open()
        {
            Console.WriteLine("已经打开当前文件夹");
        }
    }

    public class Program
    {
        static void Main()
        {
            //这是安全的组合模式
            Folder myword = new Word();

            myword.Open();//打开文件，处理文件


            Folder myfolder = new NextFolder();
            myfolder.Open();//打开文件夹

            //此处要是用增加和删除功能，需要转型的操作，否则不能使用
            ((SonFolder)myfolder).Add(new NextFolder());//成功增加文件或者文件夹
            ((SonFolder)myfolder).Remove(new NextFolder());//成功删除文件或者文件夹

            Console.Read();
        }
    }
}
```


## 实现要点
1. Composite模式采用树形结构来实现普遍存在的对象容器，从而将“一对多”的关系转化为“一对一”的关系，使得客户代码可以一致地处理对象和对象容器，无需关心处理的是单个的对象，还是组合的对象容器。
2. 将“客户代码与复杂的对象容器结构”解耦是Composite模式的核心思想，解耦之后，客户代码将与纯粹的抽象接口——而非对象容器的复杂内部实现结构——发生依赖关系，从而更能“应对变化”。
3. Composite模式中，是将“Add和Remove等和对象容器相关的方法”定义在“表示抽象对象的Component类”中，还是将其定义在“表示对象容器的Composite类”中，是一个关乎“透明性”和“安全性”的两难问题，需要仔细权衡。这里有可能违背面向对象的“单一职责原则”，但是对于这种特殊结构，这又是必须付出的代价。ASP.Net控件的实现在这方面为我们提供了一个很好的示范。
4. Composite模式在具体实现中，可以让父对象中的子对象反向追朔；如果父对象有频繁的遍历需求，可使用缓存技巧来改善效率。

## 优缺点
### 优点
1. 组合模式使得客户端代码可以一致地处理对象和对象容器，无需关系处理的单个对象，还是组合的对象容器。
2. 将”客户代码与复杂的对象容器结构“解耦。
3. 可以更容易地往组合对象中加入新的构件。

### 缺点
- 使得设计更加复杂。客户端需要花更多时间理清类之间的层次关系。（这个是几乎所有设计模式所面临的问题）。

## 使用场景
1. 需要表示一个对象整体或部分的层次结构。
2. 希望用户忽略组合对象与单个对象的不同，用户将统一地使用组合结构中的所有对象。