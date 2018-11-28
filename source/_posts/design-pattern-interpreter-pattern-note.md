---
title: 设计模式之解释器模式
date: 2018-11-28 21:41:40
categories: "设计模式"
tags:
  - 行为型
  - 设计模式
---

# 简介
解释器模式（Interpreter Pattern）提供了评估语言的语法或表达式的方式，它属于行为型模式。这种模式实现了一个表达式接口，该接口解释一个特定的上下文。这种模式被用在 SQL 解析、符号处理引擎等。

# 动机
在软件构建过程中，如果某一特定领域的问题比较复杂，类似的模式不断重复出现，如果使用普通的编程方式来实现将面临非常频繁的变化。在这种情况下，将特定领域的问题表达为某种语法规则下的句子，然后构建一个解释器来解释这样的句子，从而达到解决问题的目的。

# 意图
给定一个语言，定义它的文法的一种表示，并定义一种解释器，这个解释器使用该表示来解释语言中的句子。　　

# 结构图
![interpreter-pattern](interpreter-pattern.png)

# 模式组成
## 抽象表达式(AbstractExpression)
定义解释器的接口，约定解释器的解释操作。其中的`Interpret`接口，正如其名字那样，它是专门用来解释该解释器所要实现的功能。

## 终结符表达式(Terminal Expression)
实现了抽象表达式角色所要求的接口，主要是一个`Interpret()`方法；文法中的每一个终结符都有一个具体终结表达式与之相对应。比如有一个简单的公式`R=R1+R2`，在里面`R1`和`R2`就是终结符，对应的解析`R1`和`R2`的解释器就是终结符表达式。

## 非终结符表达式(Nonterminal Expression)
文法中的每一条规则都需要一个具体的非终结符表达式，非终结符表达式一般是文法中的运算符或者其他关键字，比如公式`R=R1+R2`中，`+`就是非终结符，解析`+`的解释器就是一个非终结符表达式。

## 环境角色(Context)
这个角色的任务一般是用来存放文法中各个终结符所对应的具体值，比如`R=R1+R2`，我们给`R1`赋值100，给`R2`赋值200。这些信息需要存放到环境角色中，很多情况下我们使用`Map`来充当环境角色就足够了。

## 客户端（Client）
指的是使用解释器的客户端，通常在这里将按照语言的语法做的表达式转换成使用解释器对象描述的抽象语法树，然后调用解释操作。

# 代码实现
```csharp
namespace InterpreterPattern
{
    // 抽象表达式
    public abstract class Expression
    {
        protected Dictionary<string, int> table = new Dictionary<string, int>(9);

        protected Expression()
        {
            table.Add("一", 1);
            table.Add("二", 2);
            table.Add("三", 3);
            table.Add("四", 4);
            table.Add("五", 5);
            table.Add("六", 6);
            table.Add("七", 7);
            table.Add("八", 8);
            table.Add("九", 9);
        }

        public virtual void Interpreter(Context context)
        {
            if (context.Statement.Length == 0)
            {
                return;
            }

            foreach (string key in table.Keys)
            {
                int value = table[key];

                if (context.Statement.EndsWith(key + GetPostFix()))
                {
                    context.Data += value * this.Multiplier();
                    context.Statement = context.Statement.Substring(0, context.Statement.Length - this.GetLength());
                }
                if (context.Statement.EndsWith("零"))
                {
                    context.Statement = context.Statement.Substring(0, context.Statement.Length - 1);
                }
            }
        }

        public abstract string GetPostFix();

        public abstract int Multiplier();

        //这个可以通用，但是对于个位数字例外，所以用虚方法
        public virtual int GetLength()
        {
            return this.GetPostFix().Length + 1;
        }
    }

    //个位表达式
    public sealed class GeExpression : Expression
    {
        public override string GetPostFix()
        {
            return "";
        }

        public override int Multiplier()
        {
            return 1;
        }

        public override int GetLength()
        {
            return 1;
        }
    }

    //十位表达式
    public sealed class ShiExpression : Expression
    {
        public override string GetPostFix()
        {
            return "十";
        }

        public override int Multiplier()
        {
            return 10;
        }
    }

    //百位表达式
    public sealed class BaiExpression : Expression
    {
        public override string GetPostFix()
        {
            return "百";
        }

        public override int Multiplier()
        {
            return 100;
        }
    }

    //千位表达式
    public sealed class QianExpression : Expression
    {
        public override string GetPostFix()
        {
            return "千";
        }

        public override int Multiplier()
        {
            return 1000;
        }
    }

    //万位表达式
    public sealed class WanExpression : Expression
    {
        public override string GetPostFix()
        {
            return "万";
        }

        public override int Multiplier()
        {
            return 10000;
        }

        public override void Interpreter(Context context)
        {
            if (context.Statement.Length == 0)
            {
                return;
            }

            ArrayList tree = new ArrayList();

            tree.Add(new GeExpression());
            tree.Add(new ShiExpression());
            tree.Add(new BaiExpression());
            tree.Add(new QianExpression());

            foreach (string key in table.Keys)
            {
                if (context.Statement.EndsWith(GetPostFix()))
                {
                    int temp = context.Data;
                    context.Data = 0;

                    context.Statement = context.Statement.Substring(0, context.Statement.Length - this.GetLength());

                    foreach (Expression exp in tree)
                    {
                        exp.Interpreter(context);
                    }
                    context.Data = temp + context.Data * this.Multiplier();
                }
            }
        }
    }

    //亿位表达式
    public sealed class YiExpression : Expression
    {
        public override string GetPostFix()
        {
            return "亿";
        }

        public override int Multiplier()
        {
            return 100000000;
        }

        public override void Interpreter(Context context)
        {
            ArrayList tree = new ArrayList();

            tree.Add(new GeExpression());
            tree.Add(new ShiExpression());
            tree.Add(new BaiExpression());
            tree.Add(new QianExpression());

            foreach (string key in table.Keys)
            {
                if (context.Statement.EndsWith(GetPostFix()))
                {
                    int temp = context.Data;
                    context.Data = 0;
                    context.Statement = context.Statement.Substring(0, context.Statement.Length - this.GetLength());

                    foreach (Expression exp in tree)
                    {
                        exp.Interpreter(context);
                    }
                    context.Data = temp + context.Data * this.Multiplier();
                }
            }
        }
    }

    //环境上下文
    public sealed class Context
    {
        private string _statement;
        private int _data;

        public Context(string statement)
        {
            this._statement = statement;
        }

        public string Statement
        {
            get { return this._statement; }
            set { this._statement = value; }
        }

        public int Data
        {
            get { return this._data; }
            set { this._data = value; }
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            string roman = "五亿七千三百零二万六千四百五十二";
            //分解：((五)亿)((七千)(三百)(零)(二)万)
            //((六千)(四百)(五十)(二))

            Context context = new Context(roman);
            ArrayList tree = new ArrayList();

            tree.Add(new GeExpression());
            tree.Add(new ShiExpression());
            tree.Add(new BaiExpression());
            tree.Add(new QianExpression());
            tree.Add(new WanExpression());
            tree.Add(new YiExpression());

            foreach (Expression exp in tree)
            {
                exp.Interpreter(context);
            }

            Console.Write(context.Data);

            Console.Read();
        }
    }
}
```

# 实现要点
使用Interpreter模式来表示文法规则，从而可以使用面向对象技巧方便地“扩展”文法。

Interpreter模式比较适合简单的文法表示，对于复杂的文法表示，Interpreter模式会产生比较大的类层次结构，需要求助于语法分析生成器这样的标准工具。

# 优缺点

## 优点
1. 易于改变和扩展文法。
2. 每一条文法规则都可以表示为一个类，因此可以方便地实现一个简单的语言。
3. 实现文法较为容易。在抽象语法树中每一个表达式节点类的实现方式都是相似的，这些类的代码编写都不会特别复杂，还可以通过一些工具自动生成节点类代码。
4. 增加新的解释表达式较为方便。如果用户需要增加新的解释表达式只需要对应增加一个新的终结符表达式或非终结符表达式类，原有表达式类代码无须修改，符合“开闭原则”

## 缺点
1. 对于复杂文法难以维护。在解释器模式中，每一条规则至少需要定义一个类，因此如果一个语言包含太多文法规则，*类的个数将会急剧增加，导致系统难以管理和维护*，此时可以考虑使用语法分析程序等方式来取代解释器模式。
2. 执行效率较低。由于在解释器模式中使用了大量的循环和递归调用，因此在解释较为复杂的句子时其速度很慢，而且代码的调试过程也比较麻烦。

# 使用场景

Interpreter模式的应用场合是Interpreter模式应用中的难点，只有满足*业务规则频繁变化，且类似的模式不断重复出现，并且容易抽象为语法规则的问题*才适合使用Interpreter模式。

1. 当一个语言需要解释执行，并可以将该语言中的句子表示为一个抽象语法树的时候，可以考虑使用解释器模式（如XML文档解释、正则表达式等领域）
2. 一些重复出现的问题可以用一种简单的语言来进行表达。
3. 一个语言的文法较为简单.
4. 当执行效率不是关键和主要关心的问题时可考虑解释器模式（注：高效的解释器通常不是通过直接解释抽象语法树来实现的，而是需要将它们转换成其他形式，使用解释器模式的执行效率并不高。）

