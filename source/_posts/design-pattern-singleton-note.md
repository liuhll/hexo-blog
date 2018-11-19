---
title: 设计模式之单例模式
date: 2018-10-11 21:52:00
categories: "设计模式"
tags:
- 单例模式
- 创建性
- 设计模式
---

## 介绍
确保一个类只有一个实例,并提供一个全局访问点。
![单例模式](单例模式.png)

## 为什么要用单列模式
单例模式的使用自然是当我们的系统中某个对象只需要一个实例的情况，例如:操作系统中只能有一个任务管理器,操作文件时,同一时间内只允许一个实例对其操作等,既然现实生活中有这样的应用场景,自然在软件设计领域必须有这样的解决方案了(因为软件设计也是现实生活中的抽象)，所以也就有了单例模式。

## 适用性
1. 当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。
2. 当这个唯一实例应该是通过子类化可扩展的，并且客户应该无需更改代码就能使用一个扩展的实例时。

## 实现思路
懒汉式与饿汉式的根本区别在与是否在类内方法外创建自己的对象。
并且声明对象都需要私有化，构造方法都要私有化，这样外部才不能通过 new 对象的方式来访问。
饿汉式的话是声明并创建对象(因为他饿)，懒汉式的话只是声明对象，在调用该类的 `getinstance()` 方法时才会进行 `new` 对象。

### 单线程Singleton实现(懒汉式,线程不安全)
```csharp
/// <summary>
    /// 单例模式的实现
    /// </summary>
    public class Singleton
    {
        // 定义一个静态变量来保存类的实例
        private static Singleton uniqueInstance;

        // 定义私有构造函数，使外界不能创建该类实例
        private Singleton()
        {
        }

        /// <summary>
        /// 定义公有方法提供一个全局访问点,同时你也可以定义公有属性来提供全局访问点
        /// </summary>
        /// <returns></returns>
        public static Singleton GetInstance()
        {
            // 如果类的实例不存在则创建，否则直接返回
            if (uniqueInstance == null)
            {
                uniqueInstance = new Singleton();
            }
            return uniqueInstance;
        }
    }
```

### 多线程实现(懒汉式,线程安全)
```csharp
    /// <summary>
    /// 单例模式的实现
    /// </summary>
    public class Singleton
    {
        // 定义一个静态变量来保存类的实例
        private static Singleton uniqueInstance;

        // 定义一个标识确保线程同步
        private static readonly object locker = new object();

        // 定义私有构造函数，使外界不能创建该类实例
        private Singleton()
        {
        }

        /// <summary>
        /// 定义公有方法提供一个全局访问点,同时你也可以定义公有属性来提供全局访问点
        /// </summary>
        /// <returns></returns>
        public static Singleton GetInstance()
        {
            // 当第一个线程运行到这里时，此时会对locker对象 "加锁"，
            // 当第二个线程运行该方法时，首先检测到locker对象为"加锁"状态，该线程就会挂起等待第一个线程解锁
            // lock语句运行完之后（即线程运行完之后）会对该对象"解锁"
            // 双重锁定只需要一句判断就可以了
            if (uniqueInstance == null)
            {
                lock (locker)
                {
                    // 如果类的实例不存在则创建，否则直接返回
                    if (uniqueInstance == null)
                    {
                        uniqueInstance = new Singleton();
                    }
                }
            }
            return uniqueInstance;
        }
    }

```

### 静态实现(饿汉式)

```csharp
public class Singleton
{
    public static readonly Singleton instance = new Singleton();
    private Singleton() { }
}
```

与下面的实现方式相同
```csharp
class Singleton
{
    public static readonly Singleton instance;
    static Singleton()
    {
        instance = new Singleton();
    }
     private Singleton() { }
}
```