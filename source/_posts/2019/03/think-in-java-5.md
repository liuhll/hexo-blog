---
title: Java编程思想学习笔记五：控制执行流程
date: 2019-03-16 21:00:27
categories: "java"
tags:
- java
- java编程思想
---

# true和false
关系操作符构造的条件语句如`==`的返回值是`true`和`false`，Java中不允许将一个数字作为布尔值使用。
# if-else
`if else语句`与其它语言的相同，其中`else`是可选的。`if else`用来实现多种条件下的执行。

# 迭代
`while`、`do-while`、`for`用来控制循环，有时候将他们称为迭代语句。语句会重复执行，直到起控制作用的布尔值得到“假”的结果时停止。

# for-each
`for-each`是一种更加简洁的for语句，语法如下：
```java
for(int i:x) {
    //---
}
// x是要被访问的循环体，i是一个变量，类型int只是一种，具体的类型要与x内部的值的类型相同，这个语句的意思就是循环取x内部的值赋值给i
```
# return
`return`关键字有两个用途:
- 一方面指定一个方法返回什么值
- 另一方面它会强制结束当前方法，并返回那个值。

# break和continue
 在任何迭代语句的主体部分，都可以用`break`和`continue`控制循环的流程，其中`break`用于强制退出循环，不执行循环剩余的语句，比如一共五组数据循环到第三组`break`，那么后面两组不管了继续执行下边的数据。`continue`是停止当前的迭代，退到循环开始执行下一次迭代，比如一共五组数据执行到第三组开始的时候`continue`，那么这个循环体主体剩余的部分不执行，继续从第四组开始执行。

# 臭名昭著的go-to
`go-to`语句会破坏代码的逻辑结构，降低代码的可读性。因此不建议使用。

# switch
`switch`也被划为一种选择语句，根据整数表达式的值，从一系列语句中选择一组执行。语法如下：
```java
switch(a){
    case value1:
       //---;
        break;
    case value2:
        //---;
        break;
    default:
        //---;
}
```

# 总结
文中多数控制语句在其它语言中都通用，注意`for-each`语句的使用。
