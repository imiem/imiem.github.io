---
layout: post
title:  "Java 中的 单例"
date:   2018-05-20 21:06:05
categories: Java
tags: Java Singleton
---

* content
{:toc}



## 单例的好处
* 对于频繁使用的对象，节省 new 操作花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销。
* 由于 new 操作的次数减少，因而对系统内存的使用频率降低，这将减轻 GC 压力，缩短 GC 停顿时间。

## 单例的实现方式

### 方式一:
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
#### 缺点：
* 有可能被意外的初始化，如下代码中，如果在其他地方调用`Elvis.STATUS`，都会被导致类被初始化，但是类初始化只有一次，因此 instance 实例永远只会被调用一次，虽然这样对单例的结果没有太大的影响，但是却有可能失去单例的优点。

```java
public class Elvis {

    public static int STATUS = 1;
    
    private Elvis() {
        System.out.println("Elvis is created!");
    }

    private static final Elvis INSTANCE = new Elvis();

    public static Elvis getInstance() {
        return INSTANCE;
    }
}

```


### 方式二：

```java
public class Elvis {

    private static Elvis INSTANCE = null;

    public synchronized Elvis getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new Elvis();
        }
        return INSTANCE;
    }
}

```

#### 优点
* 充分利用了延迟加载，在真正使用时创建对象。

#### 缺点
* 并发环境下加锁，竞争激烈的场合对性能可能有影响。

### 方法三：

```java
public class Elvis {

    private static class ElvisHolder {
        private static final Elvis INSTANCE = new Elvis();
    }

    public Elvis getInstance() {
        return ElvisHolder.INSTANCE;
    }
}
```

### 参考
* Effective Java
* 实战 Java 高并发程序设计


