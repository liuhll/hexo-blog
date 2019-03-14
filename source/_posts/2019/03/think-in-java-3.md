---
title: Java编程思想学习笔记三:static关键字的四种用法
date: 2019-03-14 22:03:51
tags:
- java
- java编程思想
---

在Java中这是一个很重要的关键字，它有很多的用法，并且在某些特定的情况下使用可以优化程序的性能。本文学习`static`关键字的应用场景。在这之前了解变量的类型，变量按作用域分为成员变量和局部变量，成员变量也就是全局变量，它是在类中声明的，不属于类中任何一个方法。而局部变量是在类中的方法体中声明的，作用域只是这个方法体。接下来说一下static的作用。

# 1. 修饰成员变量
`static`最常用的功能就是修饰成员变量，被`static`修饰的成员变量称为**静态变量**也叫做**类变量**，它与普通的成员变量的区别是：对于静态变量内存中只有一个拷贝，`JVM`只为其分配一次内存，在加载类的过程中完成内存的分配。也就是说这个变量时属于这个类的，而*其它任何这个类实例化的对象都拥有同一个拷贝*。可以使用`类名.变量`的形式进行访问。而对于普通的成员变量，每创建一个对象就会产生一个拷贝，并且每创建一个对象就会分配一次内存。所以相比之下`static`常用于对象之前需要共享变量以及方便访问变量时使用。

```java
public class TestStatic {
	private String name;
	private int age;
	public void print(){
		System.out.println("Name:" + name + "---Age:" + age);
	}
	public static void main(String[] args) {
		TestStatic ts1 = new TestStatic();
		TestStatic ts2 = new TestStatic();
		ts1.name = "zhangsan";
		ts1.age = 22;
		ts2.name = "lisi";
		ts2.age = 33;
		ts1.print();
		ts2.print();
	}
}


//  以上代码是普通的成员变量的例子，对象ts1和ts2分别拥有着age 和 name两个成员变量的两个副本，彼此互相不影响，所以在实例化之后赋值，ts1能得到想要的值，ts2也能得到想要的值。有两个副本互不影响的意思就是他们指向两个不同的内存空间。

```
- 输出
![1](1.png)

```java

public class TestStatic {
	private String name;
	private static int age; // 将其用static修饰为静态成员变量
	public void print(){
		System.out.println("Name:" + name + "---Age:" + age);
	}
	public static void main(String[] args) {
		TestStatic ts1 = new TestStatic();
		TestStatic ts2 = new TestStatic();
		ts1.name = "zhangsan";
		ts1.age = 22;
		ts2.name = "lisi";
		ts2.age = 33;
		ts1.print();
		ts2.print();
	}
// 这时由于age变量变成了类变量，两个对象ts1和ts2公用一个相同的副本，所以ts1修改过后的值会被ts2重新修改。
//  此外第二个示例代码中，静态成员变量使用了对象.变量的方式进行调用，这里编译器会给出警告，使用类名.方法之后警告就会解除
```
![2](2.png)

# 2. 修饰成员方法
`static`的另一个作用是修饰成员方法，修饰成员方法的目的是使该方法可以通过类名.方法的形式进行调用，避免了创建对象的繁琐。同时对于存储空间来说，被修饰的方法没有本质上的区别，它与不被修饰的方法都属于类方法，每个对象调用的都是同一个方法。

# 3. 修饰代码块

## 对象的初始化过程
```java
class Load {
	public Load(String msg){
		System.out.println(msg);
	}
}
public class TestStatic{
	Load ld1 = new Load("普通变量1");
	Load ld2 = new Load("普通变量2");
	static Load ld3 = new Load("静态变量3");
	static Load ld4 = new Load("静态变量4");
	public TestStatic(String msg){
		System.out.println(msg);
	}
	public static void main(String[] args) {
		TestStatic ts = new TestStatic("TestStatic 初始化");
	}

```
静态成员变量最先被初始化，并且按照执行的先后顺序进行初始化。其次初始化的是成员变量，最后初始化的是构造方法。所以在创建一个对象的时候，最先被初始化的是静态成员变量。
![3](3.png)

```java

class Load {
	public Load(String msg){
		System.out.println(msg);
	}
}
public class TestStatic{
	Load ld1 = new Load("普通变量1");
	Load ld2 = new Load("普通变量2");
	static Load ld3 = new Load("静态变量3");
	static Load ld4 = new Load("静态变量4");
	public TestStatic(String msg){
		System.out.println(msg);
	}
	public static void staticFunc(){
		System.out.println("静态方法");
	}
	public static void main(String[] args) {
		TestStatic.staticFunc();
		System.out.println("@@@@@@@@@");
		TestStatic ts = new TestStatic("TestStatic 初始化");
	}
}

```
![4](4.png)
静态成员的初始化发生在创建对象之前，确切的说是在调用静态方法之前就已经被初始化了。并且，当我们创建对象的时候，原本被初始化过的静态成员变量跟静态方法没有再次被初始化。

## 静态作用域
这时我们的`static`的作用就是，修饰一段都需要被修饰为`static`的域。被`static`修饰的代码域，域中所有的内容都被当成`static`变量，且优先初始化。

```java
class Load {
	public Load(String msg){
		System.out.println(msg);
	}
}
public class TestStatic{
	Load ld1 = new Load("普通变量1");
	Load ld2 = new Load("普通变量2");
	static Load ld3;
	static Load ld4;
	static{
		ld3 = new Load("静态变量3");
		ld4 = new Load("静态变量4");
	}
	
	public TestStatic(String msg){
		System.out.println(msg);
	}
	public static void staticFunc(){
		System.out.println("静态方法");
	}
	public static void main(String[] args) {
		TestStatic.staticFunc();
		System.out.println("@@@@@@@@@");
		TestStatic ts = new TestStatic("TestStatic 初始化");
	}
}

```
![5](5.png)

# 4. 静态导入

可以导入一个带有静态方法的静态包，从而可以在当前类中直接调用包中的静态方法，就好像是自己的方法一样。
```java
//先在一个包中创建一个类并定义一个静态方法：
package com.chenxyt.java.test;
public class Printer {
	public static void print(String msg){
		System.out.println(msg);
	}


//然后在另一个包中用import static 导入这个类：
package com.chenxyt.java.practice;
import static com.chenxyt.java.test.Printer.*;
public class TestStatic{
	public static void main(String[] args) {
		print("This is TestStatic");
	}
}

```

# 总结
`static`是Java语言中的一个很重要的关键字，主要用途有三个方面修饰成员变量，修饰成员方法以及修饰代码块。使用`static`修饰的成员变量在类加载的时候就已经被初始化了，它属于类变量，不属于某个对象，所有该类的实例化对象拥有同一个静态成员变量副本，常用的用途可以用它来做**计数器**。

