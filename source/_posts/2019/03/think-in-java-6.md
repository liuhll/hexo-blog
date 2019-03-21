---
title: Java编程思想学习笔记：初始化与清理
date: 2019-03-17 13:42:31
categories: "java"
tags:
- java
- java编程思想
---

# 使用构造器确保初始化
Java中通过提供构造器，确保每个类的对象都可以得到初始化，构造器的形式为：

在类的内部定义的一个与类名相同的方法，该方法没有返回值，没有返回值并不是返回`void`，而是真正的无返回值。该方法在对象创建时会自动执行。

```java
className（）{
    //---
}
```

案例:
1. 案例1
```java
package com.chenxyt.java.practice;
public class ConstructorTest{
	ConstructorTest(){
		System.out.println("Constructor Begin");
	}
	public static void main(String[] args) {
		for(int i=0;i<5;i++){
			ConstructorTest ct = new ConstructorTest();
		}
	}
}
```
![1](1.png)

2. 可以显示的编写有参的构造方法，给类的成员变量赋值
```java

package com.chenxyt.java.practice;
public class ConstructorTest{
	private int age;
	ConstructorTest(int i){
		age=i;
	}
	public static void main(String[] args) {
		for(int i=0;i<5;i++){
			ConstructorTest ct = new ConstructorTest(i);
			System.out.println("Age=" + ct.age);
		}
	}
}
```

> **Notes**
> 如果类中只有唯一的一个带参数的构造器，那么默认的无参构造器将不可用。


# 方法重载
假如我们有多种多样的需求，想要初始化不同形态的对象，这时候我们可能需要多个构造器才能满足需求。而构造器的名字又是固定的，所以这个时候变化的就是参数类型或者是参数个数。我们把这种方法名不变，只改变参数类型或者参数个数的操作叫做方法重载。方法重载不仅支持构造器方法重载，也支持普通方法的重载。

```java

package com.chenxyt.java.practice;
public class ConstructorTest{
	private int age;
	private String name;
	ConstructorTest(){
		
	}
	ConstructorTest(int i){
		age=i;
	}
	ConstructorTest(String j){
		name=j;
	}
	ConstructorTest(int i,String j){
		age=i;
		name=j;
	}
	public static void main(String[] args) {
		ConstructorTest ct1 = new ConstructorTest();
		ConstructorTest ct2 = new ConstructorTest(22);
		ConstructorTest ct3 = new ConstructorTest("张三");
		ConstructorTest ct4 = new ConstructorTest(23,"李四");
		System.out.println("ct1 age=" + ct1.age + "---name=" + ct1.name);
		System.out.println("ct2 age=" + ct2.age + "---name=" + ct2.name);
		System.out.println("ct3 age=" + ct3.age + "---name=" + ct3.name);
		System.out.println("ct4 age=" + ct4.age + "---name=" + ct4.name);
	}
}
//输出
// ct1 age=0---name=null
// ct2 age=22---name=null
// ct3 age=0---name=张三
// ct4 age=23---name=李四

//说明
//四个不同的构造函数初始化了四个不同的对象，可以从打印结果看出，没有初始化的值int类型为0，String类型为null。
```

# 缺省构造器
如前文所述，默认的构造器，就是没有形参的构造器，作为一个类的缺省构造器。如果程序员没有显示的在代码中创建一个构造器，那么Java会自动帮你创建一个无参构造器来完成初始化。当然如果程序员显示的创建了构造函数，那么Java就不会给你创建缺省构造器了。

# this关键字

## 表示类的当前对象的引用

```java
package com.chenxyt.java.practice;
public class ThisTest{
	public ThisTest doFunc(){
		return this;
	}
	public static void main(String[] args) {
		ThisTest tt = new ThisTest().doFunc();
	}
```
## 表示类的成员变量
```java
package com.chenxyt.java.practice;
public class ThisTest{
	private String arg;
	ThisTest(String arg){
		this.arg = arg; //此处this.arg表明了这个变量是成员变量，与构造方法的形参做了区分。
	}
	public static void main(String[] args) {
		ThisTest tt = new ThisTest("嘻嘻");
		System.out.println(tt.arg);
	}
}
// 输出
// 嘻嘻
```

## 在构造器中调用另一个构造器，使用this带参数代替构造器的方法名
```java
package com.chenxyt.java.practice;
public class ThisTest{
	ThisTest(int i,String j){
		this(j);
		System.out.println("我是构造器1");
	}
	ThisTest(String arg){
		System.out.println("我是构造器2");
	}
	public static void main(String[] args) {
		ThisTest tt = new ThisTest(2,"嘻嘻");
	}
}

//输出
// 我是构造器2
// 我是构造器1
```
# 清理：终结处理和垃圾回收
Java中的内存清理使用的是Java自带的垃圾回收机制，但是不管怎么来说，这种方式都不是绝对安全的。Java的垃圾回收机制清理的是通过new创建的对象，而在某些特殊情况下，可能有些对象不是通过new创建的，这些对象如果不使用的时候，垃圾回收器是不能准确的清理他们的从而造成了这块特殊的内存区域一直得不到释放。

Java中提供了`finalize()`方法来处理这一部分特殊的内存区域。它的处理流程是这样的，在Java垃圾回收器启用之前，会先调用这个方法进行一些必要的回收操作。但是针对这一块特殊的区域，或者是new创建的需要被回收的对象，一般情况下只有当Java虚拟机内存快要消耗殆尽的时候，垃圾回收器才会启动，毕竟启动垃圾回收器也是需要消耗资源的，所以不可能说实时存在。所说的特殊的创建对象方式，一般是指“本地方法”使用时，也就是在Java中调用非Java代码的时候发生的。所以一般情况下是不需要使用`finalize()`方法的。

`finalize()` 通常还有另一个用法，由于它是在Java垃圾回收器启动之前执行的，所以可以用它来判断终结状态，即判断一个对象是否满足回收条件。

# 成员初始化
Java尽量保证每个变量在使用之前都进行了初始化操作，变量分为局部变量和成员变量，局部变量如果没有显示的初始化，在使用它的时候会报错，而成员变量不会，如果没有显示的初始化一个成员变量，那么它会被默认的分配一个指定的值。

# 构造器初始化
如前边对构造器的阐述，可以使用构造器来初始化类的成员变量，当对象被实例化之后，对象的成员变量会被初始化。静态成员变量只有在第一次使用它是才会被初始化，后边再次用到时不会被初始化。初始化顺序为创建对象时，先初始化这个类的静态变量，然后在堆上为这个对象分配内存，最后执行构造函数。

# 数组初始化
数据是一系列相同数据类型封装起来的序列，它的初始化可以发生在任何时候，`int[] a1`表示一个`int`类型的数组，这个数组内部所有的值都是`int`类型，a1只是这个数组的一个引用，可以显示的通过如`int[] a1={1，2，3};`的形式进行初始化。如果不能确定数组的内容或者是长度，则可以通过new的形式来创建一个数组。`int[] a = new int[20];`这种创建也只是创建了一个引用数组，直到数组中的每一个字段都有确切的值了，初始化才真正的完成。如`a[1]=3;`

使用数组我们可以构建一个变参的函数，就是当方法的参数类型和个数都不确定的时候，我们可以使用一个数组作为形参。因为`Object`类是所有类型的父类，所以这个形参数组的类型就是`Obejct`类，对于基本数据类型，因为都有对应的包装类，所以也可以转换成`Obejct`类进行使用。

```java

package com.chenxyt.java.practice;
public class DifArgTest{
	public static void printArray(Object[] args){
		for(Object obj:args){
			System.out.print(obj + "");
		}
	}
	public static void main(String[] args) {
		DifArgTest.printArray(new Object[]{"我今年",new Integer(24),"岁！"});
	}
}
// 输出
// 我今年24岁

package com.chenxyt.java.practice;
public class DifArgTest{
	public static void printArray(Object...args){
		for(Object obj:args){
			System.out.print(obj + "");
		}
	}
	public static void main(String[] args) {
		DifArgTest.printArray("我","今年",24,"岁");
	}
}
```

# 枚举类型

```java
package com.chenxyt.java.practice;
public class EnumTest{
	public enum EnumSet{
		FIRST,SECOND,THIRD
	}
	public static void main(String[] args) {
		EnumSet es = EnumSet.FIRST;
		System.out.println(es);
	}	
}
// 创建一个枚举类型，类型内部的实例值是常量，因此按照通用的命名规范进行大写。同时需要使用枚举时，可以初始化一个引用然后进行赋值

```


当我们创建枚举的时候编译器会自动为我们添加一些有用的特性，我觉得这是与其它语言相比更加完备的地方，比如它会创建一个`toString()`方法，这也就是为什么我们上边可以使用syso打印出来。编译器还会创建`ordinal()`方法，用来表示特定`enum`常量的声明顺序，以及一个`static values()`方法，该方法是一个静态的方法可以通过`enum`名字进行访问，方法的作用是按照`enmu`的声明顺序，产生一个由`enum`常量值组成的数组。

```java
package com.chenxyt.java.practice;
public class EnumTest{
	public enum EnumSet{
		FIRST,SECOND,THIRD
	}
	public static void main(String[] args) {
		for(EnumSet et:EnumSet.values()){
			System.out.println("value is:" + et + "---ordinal is:" + et.ordinal());
		}
	}	
}
//输出
// value is:FIRST---ordinal is:0"
// value is:SECOND---ordinal is:1"
// value is:THIRD---ordinal is:2"
```

`enum`看起来像是一种新的数据类型，但是实际上`enum`是一个类，并且具有自己的方法。
除了上边的特性之外，`enum`还有个更加实用的特性，由于它是一个常量集，因此可以和`switch`语句完美匹配。


# 总结

本章在Java中占有了至关重要的作用，也可以说初始化比较重要。总的来说要掌握如何进行初始化，方法的重载，重载不仅发生在构造方法中，也可以发生在普通方法。this关键的使用可以说很重要，但是便于理解。垃圾回收以及枚举只是初步的了解了一下，后续还会单独进行学习。