---
title: 设计模式之备忘录模式
date: 2018-11-27 21:22:11
categories: "设计模式"
tags:
  - 行为型
  - 设计模式
---
# 简介
备忘录模式（Memento Pattern）保存一个对象的某个状态，以便在适当的时候恢复对象。备忘录模式属于行为型模式。

一个对象肯定会有很多状态，这些状态肯定会相互转变而促进对象的发展，如果要想在某一时刻把当前对象回复到以前某一时刻的状态，这个情况用“备忘录模式”就能很好解决该问题。
![memto](memto.png)

# 动机
在软件构建过程中，某些对象的状态在转换的过程中，可能由于某种需要，要求程序能够回溯到对象之前处于某个点时的状态。如果使用一些公有接口来让其他对象得到对象的状态，便会暴露对象的细节实现。如何实现对象状态的良好保存与恢复，但同时又不会因此而破坏对象本身的封装性？

# 意图
在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态（如果没有这个关键点，其实深拷贝就可以解决问题）。这样以后就可以将该对象恢复到原先保存的状态。

# 结构图
![memto-pattern](memento-pattern.png)

# 模式组成
## 发起人角色（Originator）
记录当前时刻的内部状态，负责创建和恢复备忘录数据。负责创建一个备忘录`Memento`，用以记录当前时刻自身的内部状态，并可使用备忘录恢复内部状态。`Originator`【发起人】可以根据需要决定`Memento`【备忘录】存储自己的哪些内部状态。

## 备忘录角色（Memento）
负责存储发起人对象的内部状态，在进行恢复时提供给发起人需要的状态，并可以防止`Originator`以外的其他对象访问备忘录。备忘录有两个接口：`Caretaker`【管理角色】只能看到备忘录的窄接口，他只能将备忘录传递给其他对象。`Originator`【发起人】却可看到备忘录的宽接口，允许它访问返回到先前状态所需要的所有数据。

##管理者角色（Caretaker）
负责保存备忘录对象。负责备忘录`Memento`，不能对`Memento`的内容进行访问或者操作。

# 代码实现

```csharp
namespace MementoPattern
{
    // 联系人--需要备份的数据，是状态数据，没有操作
    public sealed class ContactPerson
    {
        //姓名
        public string Name { get; set; }

        //电话号码
        public string MobileNumber { get; set; }
    }

    // 发起人--相当于【发起人角色】Originator
    public sealed class MobileBackOriginator
    {
        // 发起人需要保存的内部状态
        private List<ContactPerson> _personList;


        public List<ContactPerson> ContactPersonList
        {
            get
            {
                return this._personList;
            }

            set
            {
                this._personList = value;
            }
        }
        //初始化需要备份的电话名单
        public MobileBackOriginator(List<ContactPerson> personList)
        {
            if (personList != null)
            {
                this._personList = personList;
            }
            else
            {
                throw new ArgumentNullException("参数不能为空！");
            }
        }

        // 创建备忘录对象实例，将当期要保存的联系人列表保存到备忘录对象中
        public ContactPersonMemento CreateMemento()
        {
            return new ContactPersonMemento(new List<ContactPerson>(this._personList));
        }

        // 将备忘录中的数据备份还原到联系人列表中
        public void RestoreMemento(ContactPersonMemento memento)
        {
            this.ContactPersonList = memento.ContactPersonListBack;
        }

        public void Show()
        {
            Console.WriteLine("联系人列表中共有{0}个人，他们是:", ContactPersonList.Count);
            foreach (ContactPerson p in ContactPersonList)
            {
                Console.WriteLine("姓名: {0} 号码: {1}", p.Name, p.MobileNumber);
            }
        }
    }

    // 备忘录对象，用于保存状态数据，保存的是当时对象具体状态数据--相当于【备忘录角色】Memeto
    public sealed class ContactPersonMemento
    {
        // 保存发起人创建的电话名单数据，就是所谓的状态
        public List<ContactPerson> ContactPersonListBack { get; private set; }

        public ContactPersonMemento(List<ContactPerson> personList)
        {
            ContactPersonListBack = personList;
        }
    }

    // 管理角色，它可以管理【备忘录】对象，如果是保存多个【备忘录】对象，当然可以对保存的对象进行增、删等管理处理---相当于【管理者角色】Caretaker
    public sealed class MementoManager
    {
        //如果想保存多个【备忘录】对象，可以通过字典或者堆栈来保存，堆栈对象可以反映保存对象的先后顺序
        //比如：public Dictionary<string, ContactPersonMemento> ContactPersonMementoDictionary { get; set; }
        public ContactPersonMemento ContactPersonMemento { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            List<ContactPerson> persons = new List<ContactPerson>()
            {
                new ContactPerson() { Name="黄飞鸿", MobileNumber = "13533332222"},
                new ContactPerson() { Name="方世玉", MobileNumber = "13966554433"},
                new ContactPerson() { Name="洪熙官", MobileNumber = "13198765544"}
            };

            //手机名单发起人
            MobileBackOriginator mobileOriginator = new MobileBackOriginator(persons);
            mobileOriginator.Show();

            // 创建备忘录并保存备忘录对象
            MementoManager manager = new MementoManager();
            manager.ContactPersonMemento = mobileOriginator.CreateMemento();

            // 更改发起人联系人列表
            Console.WriteLine("----移除最后一个联系人--------");
            mobileOriginator.ContactPersonList.RemoveAt(2);
            mobileOriginator.Show();

            // 恢复到原始状态
            Console.WriteLine("-------恢复联系人列表------");
            mobileOriginator.RestoreMemento(manager.ContactPersonMemento);
            mobileOriginator.Show();

            Console.Read();
        }
    }
}
```

# 实现要点
备忘录（`Memento`）存储原发器（`Originator`）对象的内部状态，在需要时恢复原发器状态。`Memento`模式适用于*由原发器管理，却又必须存储在原发器之外的信息*。

在实现`Memento`模式中，要防止原发器以外的对象访问备忘录对象。备忘录对象有两个接口，一个为原发器使用的宽接口；一个为其他对象使用的窄接口。在实现`Memento`模式时，要考虑拷贝对象状态的效率问题，如果对象开销比较大，可以采用某种增量式改变（即只记住改变的状态）来改进Memento模式。

我们也可以用序列化的方式实现备忘录。序列化之后，我们可以把它临时性保存到数据库、文件、进程内、进程外等地方。

# 优缺点

## 优点
1. 如果某个操作错误地破坏了数据的完整性，此时可以使用备忘录模式将数据恢复成原来正确的数据。
2. 备份的状态数据保存在发起人角色之外，这样发起人就不需要对各个备份的状态进行管理。而是由备忘录角色进行管理，而备忘录角色又是由管理者角色管理，符合单一职责原则。
3. 提供了一种状态恢复的实现机制，使得用户可以方便地回到一个特定的历史步骤，当新的状态无效或者存在问题时，可以使用先前存储起来的备忘录将状态复原。
4. 实现了信息的封装，一个备忘录对象是一种原发器对象的表示，不会被其他代码改动，这种模式简化了原发器对象，备忘录只保存原发器的状态，采用堆栈来存储备忘录对象可以实现多次撤销操作，可以通过在负责人中定义集合对象来存储多个备忘录。
5. 本模式简化了发起人。发起人不再需要管理和保存其内部状态的一个个版本，客户端可以自行管理他们所需要的这些状态的版本。
6. 当发起人角色的状态改变的时候，有可能这个状态无效，这时候就可以使用暂时存储起来的备忘录将状态复原。

## 缺点
1. 在实际的系统中，可能需要维护多个备份，需要额外的资源，这样对资源的消耗比较严重。资源消耗过大，如果类的成员变量太多，就不可避免占用大量的内存，而且每保存一次对象的状态都需要消耗内存资源，如果知道这一点大家就容易理解为什么一些提供了撤销功能的软件在运行时所需的内存和硬盘空间比较大了。
2. 如果发起人角色的状态需要完整地存储到备忘录对象中，那么在资源消耗上面备忘录对象会很昂贵。
3. 当负责人角色将一个备忘录 存储起来的时候，负责人可能并不知道这个状态会占用多大的存储空间，从而无法提醒用户一个操作是否很昂贵。
4. 当发起人角色的状态改变的时候，有可能这个协议无效。如果状态改变的成功率不高的话，不如采取“假如”协议模式。

# 使用场景
1. 如果系统需要提供回滚操作时，使用备忘录模式非常合适。例如文本编辑器的Ctrl+Z撤销操作的实现，数据库中事务操作。
2. 保存一个对象在某一个时刻的状态或部分状态，这样以后需要时它能够恢复到先前的状态。
3. 如果用一个接口来让其他对象得到这些状态，将会暴露对象的实现细节并破坏对象的封装性，一个对象不希望外界直接访问其内部状态，通过负责人可以间接访问其内部状态。
4. 有时一些发起人对象的内部信息必须保存在发起人对象以外的地方，但是必须要由发起人对象自己读取，这时，使用备忘录模式可以把复杂的发起人内部信息对其他的对象屏蔽起来，从而可以恰当地保持封装的边界。

# 封装性
为了确保备忘录的封装性，除了原发器外，其他类是不能也不应该访问备忘录类的，在实际开发中，原发器与备忘录之间的关系是非常特殊的，它们要分享信息而不让其他类知道，实现的方法因编程语言的不同而不同。

# 多备份实现
1. 在负责人中定义一个集合对象来存储多个状态，而且可以方便地返回到某一历史状态。
2. 在备份对象时可以做一些记号，这些记号称为检查点(Check Point)。在使用HashMap等实现时可以使用Key来设置检查点。