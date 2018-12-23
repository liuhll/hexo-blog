---
title: 设计模式之迭代器模式
date: 2018-11-21 22:28:42
categories: "设计模式"
tags:
  - 行为型
  - 设计模式
---

# 简介
迭代器模式（Iterator Pattern）是 Java 和 .Net 编程环境中非常常用的设计模式。这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。
迭代器模式属于行为型模式。

# 动机
在软件构建过程中，集合对象内部结构常常变化各异。但对于这些集合对象，我们希望在不暴露其内部结构的同时，可以让外部客户代码透明地访问其中包含的元素；同时这种“透明遍历”也为“同一种算法在多种集合对象上进行操作”提供了可能。
使用面向对象技术将这种遍历机制抽象为“迭代器对象”为“应对变化中的集合对象”提供了一种优雅的方式

# 意图
提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露该对象的内部表示。

# 结构图
![Iterator-Pattern](Iterator-Pattern.png)

# 组成
## 1.抽象迭代器(Iterator)
抽象迭代器定义了访问和遍历元素的接口，一般声明如下方法：用于获取第一个元素的`first()`，用于访问下一个元素的`next()`，用于判断是否还有下一个元素的`hasNext()`，用于获取当前元素的`currentItem()`，在其子类中将实现这些方法。
## 2.具体迭代器(ConcreteIterator)
具体迭代器实现了抽象迭代器接口，完成对集合对象的遍历，同时在对聚合进行遍历时跟踪其当前位置。

## 3.抽象聚合类(Aggregate)
抽象聚合类用于存储对象，并定义创建相应迭代器对象的接口，声明一个`createIterator()`方法用于创建一个迭代器对象。

## 4.具体聚合类(ConcreteAggregate)
具体聚合类实现了创建相应迭代器的接口，实现了在抽象聚合类中声明的`createIterator()`方法，并返回一个与该具体聚合相对应的具体迭代器`ConcreteIterator`实例。

# 代码实现

# 实现要点
```csharp
namespace 迭代器模式的实现
{
    // 部队队列的抽象聚合类--该类型相当于抽象聚合类Aggregate
    public interface ITroopQueue
    {
        Iterator GetIterator();
    }
 
    // 迭代器抽象类
    public interface Iterator
    {
        bool MoveNext();
        Object GetCurrent();
        void Next();
        void Reset();
    }
 
    //部队队列具体聚合类--相当于具体聚合类ConcreteAggregate
    public sealed class ConcreteTroopQueue:ITroopQueue
    {
        private string[] collection;

        public ConcreteTroopQueue()
        {
            collection = new string[] { "黄飞鸿","方世玉","洪熙官","严咏春" };
        }
 
        public Iterator GetIterator()
        {
            return new ConcreteIterator(this);
        }
 
        public int Length
        {
            get { return collection.Length; }
        }
 
        public string GetElement(int index)
        {
            return collection[index];
        }
    }
 
    // 具体迭代器类
    public sealed class ConcreteIterator : Iterator
    {
        // 迭代器要集合对象进行遍历操作，自然就需要引用集合对象
        private ConcreteTroopQueue _list;
        private int _index;
 
        public ConcreteIterator(ConcreteTroopQueue list)
        {
            _list = list;
            _index = 0;
        }
 
        public bool MoveNext()
        {
            if (_index < _list.Length)
            {
                return true;
            }
            return false;
        }
 
        public Object GetCurrent()
        {
            return _list.GetElement(_index);
        }
 
        public void Reset()
        {
            _index = 0;
        }
 
        public void Next()
        {
            if (_index < _list.Length)
            {
                _index++;
            }
 
        }
    }
 
    // 客户端（Client）
    class Program
    {
        static void Main(string[] args)
        {
            Iterator iterator;
            ITroopQueue list = new ConcreteTroopQueue();
            iterator = list.GetIterator();
 
            while (iterator.MoveNext())
            {
                string ren = (string)iterator.GetCurrent();
                Console.WriteLine(ren);
                iterator.Next();
            }
 
            Console.Read();
        }
    }
}
```

# 实现要点
1. 迭代抽象：访问一个聚合对象的内容而无需暴露它的内部表示。
2. 迭代多态：为遍历不同的集合结构提供一个统一的接口，从而支持同样的算法在不同的集合结构上进行操作。
3. 迭代器的健壮性考虑：遍历的同时更改迭代器所在的集合结构，会导致问题。

# 优缺点

## 优点
1. 迭代器模式使得访问一个聚合对象的内容而无需暴露它的内部表示，即迭代抽象。
2. 迭代器模式为遍历不同的集合结构提供了一个统一的接口，从而支持同样的算法在不同的集合结构上进行操作

## 缺点
1. 迭代器模式在遍历的同时更改迭代器所在的集合结构会导致出现异常。所以使用foreach语句只能在对集合进行遍历，不能在遍历的同时更改集合中的元素。

# 使用场景
1. 访问一个聚合对象的内容而无需暴露它的内部表示。
2. 支持对聚合对象的多种遍历。
3. 为遍历不同的聚合结构提供一个统一的接口(即, 支持多态迭代)。