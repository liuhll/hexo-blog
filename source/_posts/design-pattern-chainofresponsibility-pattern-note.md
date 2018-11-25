---
title:  设计模式之责任链模式
date: 2018-11-25 21:49:35
categories: "设计模式"
tags:
  - 行为型
  - 设计模式
---

# 简介 
顾名思义，责任链模式（**Chain of Responsibility Pattern**）为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。
在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

# 动机
在软件构建过程中，一个请求可能被多个对象处理，但是每个请求在运行时只能有一个接受者，如果显示指定，将必不可少地带来请求发送者与接受者的紧耦合。如何使请求的发送者不需要指定具体的接受者，让请求的接受者自己在运行时决定来处理请求，从而使两者解耦。

# 意图
避免请求发送者与接收者耦合在一起，让多个对象都有可能接受请求，将这些对象连接成一条链，并且沿着这条链传递请求，知道有对象处理它为止。　　

# 结构图
![ChainOfResponsibilityPattern](ChainOfResponsibilityPattern.png)

# 模式的组成
## 抽象处理者角色（Handler）
抽象处理者定义了一个处理请求的接口，它一般设计为抽象类，由于不同的具体处理者处理请求的方式不同，因此在其中定义了抽象请求处理方法。因为每一个处理者的下家还是一个处理者，因此在抽象处理者中定义了一个自类型的对象，作为其对下家的引用。*通过该引用，处理者可以连成一条链*。

## 具体处理者角色（ConcreteHandler）
具体处理者是抽象处理者的子类，它可以处理用户请求，在具体处理者类中实现了抽象处理者中定义的抽象处理方法，在处理请求之前需要进行判断，看是否有相应的处理权限，如果可以处理请求就处理它，否则将请求转发给后继者；在具体处理者中可以访问链中下一个对象，以便请求的转发。

# 代码实现

```csharp
namespace ChainOfResponsibility
{
    // 采购请求
    public sealed class PurchaseRequest
    {
        // 金额
        public double Amount { get; set; }

        // 产品名字
        public string ProductName { get; set; }

        public PurchaseRequest(double amount, string productName)
        {
            Amount = amount;
            ProductName = productName;
        }
    }
 
    //抽象审批人,Handler---相当于“抽象处理者角色”
    public abstract class Approver
    {
        //下一位审批人，由此形成一条链
        public Approver NextApprover { get; set; }

        //审批人的名称
        public string Name { get; set; }

        public Approver(string name)
        {
            this.Name = name;
        }

        //处理请求
        public abstract void ProcessRequest(PurchaseRequest request);
    }
 
    //部门经理----相当于“具体处理者角色” ConcreteHandler
    public sealed class Manager : Approver
    {
        public Manager(string name): base(name){ }
 
        public override void ProcessRequest(PurchaseRequest request)
        {
            if (request.Amount <= 10000.0)
            {
                Console.WriteLine("{0} 部门经理批准了对原材料{1}的采购计划！", this.Name, request.ProductName);
            }
            else if (NextApprover != null)
            {
                NextApprover.ProcessRequest(request);
            }
        }
    }
 
    //财务经理---相当于“具体处理者角色”ConcreteHandler
    public sealed class FinancialManager : Approver
    {
        public FinancialManager(string name): base(name){ }

        public override void ProcessRequest(PurchaseRequest request)
        {
            if (request.Amount > 10000.0 && request.Amount <= 50000.0)
            {
                Console.WriteLine("{0} 财务经理批准了对原材料{1}的采购计划！", this.Name, request.ProductName);
            }
            else if (NextApprover != null)
            {
                NextApprover.ProcessRequest(request);
            }
        }
    }
 
    //总裁---相当于“具体处理者角色” ConcreteHandler
    public sealed class CEO :Approver
    {
        public CEO(string name): base(name){ }

        public override void ProcessRequest(PurchaseRequest request)
        {
            if (request.Amount > 50000.0 && request.Amount < 300000.0)
            {
                Console.WriteLine("{0} 总裁批准了对原材料 {1} 的采购计划！", this.Name, request.ProductName);
            }
            else
            {
                Console.WriteLine("这个采购计划的金额比较大，需要一次董事会会议讨论才能决定！");
            }
        }
    }
 
    class Program
    {
        static void Main(string[] args)
        {
            PurchaseRequest requestDao = new PurchaseRequest(8000.0, "单刀5把");
            PurchaseRequest requestHuaJi = new PurchaseRequest(10000.0, "10把方天画戟");
            PurchaseRequest requestJian = new PurchaseRequest(80000.0, "5把金丝龙鳞闪电劈");
 
            Approver manager = new Manager("黄飞鸿");
            Approver financial = new FinancialManager("黄麒英");
            Approver ceo = new CEO("十三姨");
 
            // 设置职责链
            manager.NextApprover = financial;
            financial.NextApprover = ceo;
 
            // 处理请求
            manager.ProcessRequest(requestDao);
            manager.ProcessRequest(requestHuaJi);
            manager.ProcessRequest(requestJian);

            Console.ReadLine();
        }
    }
}
```
# 实现要点
**Chain of Responsibility**模式的应用场合在于*一个请求可能有多个接受者，但是最后真正的接受者只有一个*，只有这时候请求发送者与接受者的耦合才有可能出现**变化脆弱**的症状，职责链的目的就是将二者解耦，从而更好地应对变化。

应用了**Chain of Responsibility**模式后，对象的职责分派将更具灵活性。我们可以在运行时动态添加/修改请求的处理职责。

当我们要新增一个`DHandler`处理请求，就不需再改原来的代码了，遵从了开放封闭原则。这样我们的程序就更赋予变化，更有变化的抵抗力。`Handler`类本身继承自`BaseHandler`类型，又包含了一个`BaseHandler`类型的对象，这点类似`Decorator`模式。

如果请求传递到职责链的末尾仍得不到处理，应该有一个合理的缺省机制。这也是每一个接受对象的责任，而不是发出请求的对象的责任。

# 优缺点

## 优点
1. 降低耦合度：职责链模式使得一个对象无需知道是其他哪一个对象处理其请求。对象仅需知道该请求会被处理即可，接受者和发送者都没有对方的明确信息，且链中的对象不需要知道链的结构，有客户端负责链的创建。

2. 可简化对象的相互连接：接受者对象仅需维持一个指向其后继者的引用，而不需维持它对所有的候选处理者的引用。

3. 增强给对象指派职责的灵活性：在给对象分派职责时，职责链可以给我们带来更多的灵活性。可以通过在运行时对该连进行动态的增加或修改处理一个请求的职责。

4. 增加新的请求处理类很方便：在系统中增加一个新的请求处理者无需修改原有系统的代码，只需要在客户端重新建链即可，从这一点看来是符合“开闭原则”的。

## 缺点
1. 在找到正确的处理对象之前，所有的条件判定都要执行一遍，当责任链过长时，可能会引起性能的问题。
2. 可能导致某个请求不被处理。
3. 客户端需要组装这个链条，耦合了客户端和链条的组成结构，可以把这个在客户端的组合动作提到外面，通过配置来做，会更好点。

# 使用场景 
1. 一个系统的审批需要多个对象才能完成处理的情况下，例如请假系统等。
2. 代码中存在多个`if-else`语句的情况下，此时可以考虑使用责任链模式来对代码进行重构
3. 有多个对象可以处理同一个请求，具体哪个对象处理该请求有运行时刻自动确定。客户端只需将请求提交到链上，无须关心请求的处理对象是谁以及它是如何处理的。
4. 不明确指定接受者的情况下，向多个对象中的一个提交一个请求。请求的发送者与请求者解耦，请求将沿着链进行传递，寻求响应的处理者。
5. 可动态指定一组对象处理请求。客户端可以动态创建职责链来处理请求，还可以动态改变链中处理者之间的先后次序
