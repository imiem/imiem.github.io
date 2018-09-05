---
layout: post
title:  "zookeeper实现分布式锁"
date:   2018-07-22 21:06:05
categories: zookeeper
tags: zookeeper
---

* content
{:toc}



>> 分布式锁是控制分布式系统之间**同步访问共享资源**的一种方式。如果在不同主机间共享了一个或一组资源，访问这些资源的时候，为了保证一致性，就需要使用分布式锁。

### 为什么要使用分布式锁？
一般情况下，在实际的开发中会使用数据库固有的排他性来实现不同程度的互斥，但是目前大部分的业务的瓶颈却是在数据库上，如果再加上行锁，表锁，数据库的性能变得更低。


### 排他锁
#### 什么是排他锁（Exclusive Locks，简称 X 锁）？
如果事务 T 对对象 O 加上了排他锁，那么整个加锁期间，只有 T 能对 O 进行读取和更新操作，知道 T 释放了排他锁。

#### 定义排他锁

#### 获取排他锁

#### 释放排他锁

#### 整体流程
[<img src="{{ site.baseurl }}/images/zookeeper-exclusivve-locks-process.png" alt="" style="width: 100%px;"/>]({{ site.baseurl/images/zookeeper-exclusivve-locks-process.png}}/)




