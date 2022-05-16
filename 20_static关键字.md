---
title: static关键字
date: 2020-10-15 15:08:35
tags: 
	- Java 
	- String类
	- 常用类
categories:
	- Java
cover:
	https://gitee.com/jasonM4A1/pictureHost/raw/master/20201017133432.jpg 
---

## 概述

`static`关键字可以用来修饰成员变量和成员方法，被修饰的成员是属于类的，而不是单单属于某个对象。也就是说`static`修饰的变量和方法等，不能用对象来调用。

1. 当`static`修饰成员变量时，被修饰的变量称为**静态变量**。该类的每个对象都共享同一个静态变量。任何对象都可以更改该静态变量的值，也可以在不创建该类对象的情况下对静态变量进行操作。

2. 当`static`修饰成员方法时，该方法称为**静态方法**。静态方法在声明中有`static`，建议使用类名来调用，而不需要创建类的对象。

3. 当`static`修饰代码块时，该代码块被称为**静态代码块**。静态代码块定义在类中，方法外，是使用`static`修饰的代码块｛｝。随着类的加载而执行且执行一次，优先于main方法和构造方法的执行。

## 格式

### 静态变量

~~~java
//static 数据类型 变量名
static String name;
~~~

### 静态方法

~~~java
//修饰符 static 返回值 方法名(参数列表) {}
public static void main(String[] args){}
~~~

### 静态代码块

定义在成员位置（类中，方法外），使用`static`修饰的代码块｛｝。随着类的加载而执行且执行一次，优先于main方法和构造方法的执行。

~~~java
package review;

public class Test {
    static {
        System.out.println("静态代码块");
    }

    public static void main(String[] args) {
        System.out.println("main方法");
    }
}
//执行结果:
//静态代码块
//main方法
~~~

## 注意

1. 静态方法可以直接访问静态变量和静态方法，但不能直接访问普通成员变量和成员方法。
2. 成员方法也可以直接访问静态变量或静态方法。
3. 静态方法中，不能使用**this**关键字。

