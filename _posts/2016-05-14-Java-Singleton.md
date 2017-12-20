---
layout: post
title:  "Java 中的 单例"
date:   2016-04-03 14:06:05
categories: Java
tags: Java Singleton
---

* content
{:toc}

本文中的例子来自Effective Java





## 单例的三种简单方式

### 方式一：

```java
public class Elvis {
    public static final  Elvis INSTANCE = new Elvis();
    private Elvis(){
        //
    }
}

```

### 方法二：
公有的成员是个静态成员方法：

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }
}

```

### 方法三：
绝对防止多次实例化，即使是在面对复杂的序列化或者反射攻击的时候。

```java
public enum Elvis {
    INSTANCE;
}

```