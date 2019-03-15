---
title: Java编程思想学习笔记四:操作符
date: 2019-03-15 21:08:39
tags:
- java
- java编程思想
---

# 实现更新简单的打印语句

```java
//打印语句
System.out.println("Hello World");
```
结合上一篇学习static的文章可知，打印语句如果在一个类中多次调用的时候，我们可以把这个语句抽出来写成静态方法，然后通过导入静态包的方式调用这个打印方法。

`test`包下的`Printer`类：

```java
package com.chenxyt.java.test;
public class Printer {
	public static void print(String msg){
		System.out.println(msg);
	}
}

//practice包下的TestStatic类：
package com.chenxyt.java.practice;
import static com.chenxyt.java.test.Printer.*;
public class TestStatic{
	public static void main(String[] args) {
		print("This is TestStatic");
	}
}

```

# java操作符
Java与其他语言一样，支持加号，正号，减号，负号，乘除号等操作符，同时`=`，`==`和`！=`不光可以操作基本操作类型，还可以操作所有的对象。此外，`String`类支持`+`和`+=`。

## 优先级
当一个表达式中存在多个操作符时，操作符的优先级就尤为重要了，它决定了程序运算操作执行的先后顺序。Java中的计算顺序与其它语言的基本相同，先计算乘除，再计算加减，有括号的先计算括号里边的。`System.out.println()`语句中包含`+`的操作符号，简单的只是进行字符串的连接，复杂一点的就是当编译器发现`+`前边是一个`String`类型，会尝试将`+`后面的内容转换成`String`类型。

## 赋值
**赋值**操作使用的是`=`操作符，将`=`右边的值赋给左边，右值可以是任意的常数、变量或者是表达式，只有它能生成一个值即可，而左值必须是一个明确的已命名的变量，就是必须要有个物理空间来进行存储。比如，可以将一个常数赋给变量。

```java
a=4;
//但是却不能将一个变量赋值给一个常数
4=a; //这个是错误的
```
### 基本数据类型的赋值 
对于基本的数据类型，赋值操作没有引用的涉及，只是单纯的将一个值赋值给另一个值。例如`a=b`，当`b`再次被修改时，`a`不会受到影响，因为`a`与`b`相互独立。
### 对象的赋值
于对对象的赋值来说，情况却大大不同，因为我们对对象的操作是操作了对象的引用，所以当一个对象赋值给另一个对象时，实际上是拷贝了引用到左值，也就是比如c和d是指向两个不同对象的引用，当c=d时，实际发生的情况是c和d都指向了原本只有d指向的对象。而c被赋值之后，原来的引用丢失了，它曾经所指向的不再被引用的对象被垃圾回收器回收了。

```java
package com.chenxyt.java.practice;
class Tank{
	int level;
}
public class CopyTest{
	public static void main(String[] args) {
		Tank tk1 = new Tank();
		Tank tk2 = new Tank();
		tk1.level = 2;
		tk2.level = 3;
		System.out.println("tk1.level = " + tk1.level + "---tk2.level = " + tk2.level);
		tk1=tk2;
		System.out.println("tk1.level = " + tk1.level + "---tk2.level = " + tk2.level);
		tk2.level=5;
		System.out.println("tk1.level = " + tk1.level + "---tk2.level = " + tk2.level);
	}
}

//    tk1和tk2分别是两个独立的对象，它们内部有level属性，赋值之前两个对象的属性值不同，赋值操作完成之后两个对象的属性值相同，当再次更改对象tk2的level值时，预期的理想情况是不会影响tk1的值，但实际结果并非如此，tk1与tk2对象的属性相同，这与前面的分析结果相同。

```
![1](1.png)

### 别名现象
Java中这种针对对象的特殊现象叫做*别名现象*，如果想避免这种现象的话，可以使用如下操作，**赋值操作针对属性而不是对象的引用**
```java
tk1.level=tk2.level //这样操作就可以保持两个对象本身相互独立。
```
在调用方法传参的时候也是会产生别名问题。
```java
package com.chenxyt.java.practice;
class Tank{
	int level;
}
public class CopyTest{
	public static void copy(Tank tk3){
		tk3.level = 9;
	}
	public static void main(String[] args) {
		Tank tk1 = new Tank();
		tk1.level = 2;
		System.out.println("tk1.level = " + tk1.level);
		copy(tk1);
		System.out.println("tk1.level = " + tk1.level);
	}
}

// 运行结果，方法外部tk1对象的值被修改了
// 输出
// tk1.level = 2
// tk1.level = 9
```



## 算术操作符
Java中的算术操作符与其它语言基本类似，有加号（+）、减号（-）、乘号（*）、取整（/）、取余（%），同时也具有简化运算符的功能如要将x加4之后再赋值给x，则可以写成:
```java
x+=4;
```
## 自动递增和递减
Java中的递增递减操作与其它语言基本类似，递增符合（++），递减符合（--），操作目的是快速使一个整数加1或者减1如a++等同于a=a+1，这两种符号分别有两种使用方式称为“前缀式”和“后缀式”。

### 前缀式
前缀式意味着符号在变量前边

### 后缀式
后缀式意味着符号在变量后边

### 二者区别 
二者的区别，对于前缀式是先做运算再取值，如`a=1，b=++a`，此时`a`跟`b`的值都是`2`，而后缀式则是先取值再做运算，如`a=1，b=a++`，此时`b`的值为`1`，`a`的值为`2`。

## 关系操作符
关系操作符生成的是一个`boolean`（布尔）结果，它们计算操作数之间的关系，如果关系为真则结果为`true`，如果关系为假则结果为`false`。关系操作符号包括小于（`<`）、大于（`>`）、小于等于（`<=`）、大于等于（`>=`）、等于（`==`）以及不等于（`!=`）。等于和不等于适用于所有的基本数据类型，而其它的操作符不适用于boolean类型，因为它们的值为true或者false，比较大小没有意义。

操作符`==`和`!=`同样适用于操作对象，但是与基本操作类型相比有一些不同。

```java

package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		Integer itg1 = new Integer(22);
		Integer itg2 = new Integer(22);
		System.out.println(itg1==itg2);
		System.out.println(itg1!=itg2);
		int int1 = 22;
		int int2 = 22;
		System.out.println(int1==int2);
		System.out.println(int1!=int2);
	}

//输出
//false
//true
//true
//false

//原因分析
// 因为对于对象来说，“==”和“！=”比较的是对象的引用，虽然这两个对象的值相同，但是他们对象的引用并不是一个，也就是他们在内存中有两个不同的存储位置，所以不相同。所以要想比较对象的值是否相同，我们可以使用对象的equals（）方法来进行比较，它是Object类的一个通用方法，第一章中说到过所有类的父类都是Object，因此任何类的对象都可以调用这个方法。

// 对象的对比
package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		Integer itg1 = new Integer(22);
		Integer itg2 = new Integer(22);
		System.out.println(itg1.equals(itg2));
	}
}
// 输出
// true
```

```java
package com.chenxyt.java.practice;
class Oper{
	int i;
}
public class OperationTest{
	public static void main(String[] args) {
		Oper op1 = new Oper();
		Oper op2 = new Oper();
		op1.i=2;
		op2.i=2;
		System.out.println(op1.equals(op2));
	}
}
// 输出
// false
//原因分析:Object类中equals（）方法实际上是比较两个引用是否相同，就是它与“==”的本质效果是一样的，但是在第一个示例中的Integer类中，Java覆盖了这个方法，方法内容判断两个对象的类型是否相同以及值是否相同即可。而我们自己创建的Oper类并没有覆盖这个方法，所以沿用的还是Object类的方法。常见的String类也是覆盖了equals（）方法，效果与Integer类的对象相同。

```

## 逻辑操作符

Java中的逻辑操作符与其它语言基本类似，包括与（`&&`）、或（`||`）、非（`!`），不同的是Java中的逻辑操作符只能应用与布尔值之间或者是表达式结果为布尔值。

`&&`逻辑与运算符必须两边同时为真则结果为真，否则为假。`||`逻辑或运算符只要有一边为真则结果为真，`!`非运算符取对应值的对立面，即非真则假。

### 短路
对于逻辑与和逻辑或运算还有个名词叫做短路，即当两个算式做逻辑与运算时，如果左边第一个算式为假，则显然这个逻辑表达式最后的结果一定为假，那么就没有必要进行下边的表达式计算，直接结束运算，这个过程称作短路。

## 按位操作符
Java中的按位操作符与其他语言基本相似，有按位与（`&`）、按位或（`|`）、按位异或（`^`）和按位非（`~`）操作符。它们针对基本数据类型的一个比特位（`bit`）进行运算。运算规则如下，&操作符必须同时为1才为1，|操作符只有同时为0时才为0，^操作符只要有一个为1就为1，~是单目运算符，取反，若值为1则结果为0，反之为1。按位操作符除了~还可以与“=”合起来使用，如&=或者|=、^=。
对于布尔值，同样可以进行按位与、按位或和按位异或运算，但是不能进行按位非运算，并且他们不会被短路，不管第一个表达式结果是什么，都会继续运算下去。

## 移位操作符
 移位操作符操作的也是二进制的`位`，移位操作符只能用来处理整数类型。移位操作也是二元操作符，一共有三种，左移操作符（`<<`），右移操作符（`>>`），无符号右移操作符（`>>>`），左移操作符是操作符左边的数向左移动操作符右边指定的位数，低位补0，右移操作符是操作符左边的数向右移动操作符右边指定的位数，其中正数高位补0，负数高位补1。无符号右移操作符是Java独有的一种，即不管正数还是负数，右移之后高位都补0。关于移位操作有两点说明，一是高位指的是左边的位，二是对于int类型，最大长度为32位，对于long类型最大长度为64位。


## 三元操作符IF-ELSE
```java
    boolean-exp?value1 : value2;

```
```java

package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		int a = 5;
		int b = a>10?a*100:a*10;
		System.out.println("b===="+ b);
	}
// 输出
// b====50
```

## 字符串操作符+和+=
Java中可以使用`+`和`+=`连接`String`类型，并且当操作符左边你的数为`String`时，Java会试图将操作符右边的类型转换成`String`。

## 使用操作符常犯的错误
注意`=`和`==`的区别，以及逻辑运算符如（`&&`）和位运算符（`&`）的区别和前自增`++i`和后自增`i++`的区别。注意`==`和`equals()`的使用，注意别名现象。

# 类型转换操作符
Java中有跟其它语言相同的转换方式，（类型）值形式，也可以使用基本类型的包装器的转换方法进行转换。

# Java没有size of
C语言中使用size of来获取程序占用的内存字节大小，目的是确定平台的移植操作，Java中没有这个方法，也就是不需要获取这个值，因为Java在不同的平台下基本数据类型具有相同的大小，可以便捷的移植。
十七、总结

# 总结
操作符在各个语言中基本通用，熟练掌握自增自减、逻辑运算、位运算以及移位操作即可，理解`==`和`equals()`的原理，理解`别名现象`。

