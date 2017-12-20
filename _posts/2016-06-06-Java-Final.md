---
layout: post
title:  "final关键字"
date:   2016-06-06 14:06:05
categories: Java
tags: Java final 关键字
---

* content
{:toc}

> 根据上下文，final的含义有细微的差别，但通常它指的是**“这是无法改变的”**。不想做出改变可能出于两种理由：**设计和效率**。 final一般用于三种情况：变量、方法和类。




## final变量
凡是对成员变量和本地变量（在方法中和代码块中的变量）声明为final的都称为final变量，final变量经常和static关键字一块使用，作为常量。

```java
public static final String DEL_FLAG_NORMAL = "0";
public static final String DEL_FLAG_DELETE = "1";
public static final String DEL_FLAG_AUDIT = "2";
```

## final方法
final也可以声明方法。方法前面加上final关键字，代表这个方法不可以被子类的方法重写。如果你认为一个方法的功能已经**足够完整**了，子类中不需要改变的话，你可以声明此方法为final。final方法比非final方法要快，因为在编译的时候已经静态绑定了，不需要在运行时再动态绑定。

```java
class Person {
    public final String getName() {
        return "Person";
    }

}

class Man extends Person{

    @Override // 编译错误
    public String getName() {
        return "Man";
    }

}
```

## final类
使用final来修饰的类叫作final类。final类通常功能是完整的，它们不能被继承。Java中有许多类是final的，譬如String, Interger以及其他包装类。

```java
final class Person {}

// 编译错误
class Man extends Person {}

```

## final关键字的好处？
* 提高性能，JVM和Java应用都会缓存final变量。
* final变量在多线程下可以安全的进行共享，而不需要额外的同步开销。
* final关键字，JVM会对方法，变量和类方法进行优化。




## 关键点

* 一个既是static又是final的域只占据一段不能改变的存储空间。
* 对于基本类型，final使数值恒定不变；对于对象引用，final使引用恒定不变。一旦引用被final初始化执行一个对象，就无法再把它指向另一个对象。然而，对象本身是可以改变的，Java并未提供任何使对象恒定不变的途径（可以自己编写类以实现对象恒定不变的效果）。
* 根据惯例，**既是static又是final**的域（编译期常量）将用大写表示，并使用下划线分割各个单词。
* Java允许在参数列表中以声明的方式将参数指明为final，这意味着无法在方法中更改参数引用所指向的对象。
* 类中所有的private方法都隐式的指定为是final的。
* 类声明为final时，表明了你不打算继承该类，而且也不允许别人这样做。由于final类禁止继承，所以final类中所有的方法都隐式指定为final。