---
layout: post
title:  "ZooKeeper 学习"
date:   2019-12-23 19:06:05
categories: Java
tags: ZooKeeper
---

* content
{:toc}


> > > 阅读<从 Paxos 到ZooKeeper 分布式一致性原理与实践> 做的笔记




## 集群安装

* **1、修改 zoo.cfg**

  修改%ZK_HOME%/conf/zoo.cfg

  ```
  tickTime=2000
  initLimit=10
  syncLimit=5
  dataDir=/usr/local/zookeeper-3.4.5/zkData
  clientPort=2181
  server.1=s1:2888:3888
  server.2=s2:2888:3888
  server.3=s3:2888:3888
  ```

* **2、在 dataDir 中创建 myid**

  即在/usr/local/zookeeper-3.4.5/zkData下创建 myid。

  * myid 文件中只有一个数字，server.1的 myid 文件内容是 "1"。
  * myid 文件中的数字和 zoo.cfg 中 server.id=host:port:port的 id 一致。
  * id 的范围为 1~255。

* **3、为其他机器都配置上 zoo.cfg和 myid 文件。**

* **4、启动**

  在%ZK_HOME%/bin 目录下执行下面命令

  ```
  ./zkServer.sh start
  看到下面内容
  JMX enabled by default
  Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
  Starting zookeeper ... STARTED
  ```

* **5、验证**

  使用 telnet 和 stat 验证是否正常启动。

  ```
  telnet 192.168.7.30 2181
  Trying 192.168.7.30...
  Connected to 192.168.7.30.
  Escape character is '^]'.
  stat
  Zookeeper version: 3.4.5-1392090, built on 09/30/2012 17:52 GMT
  Clients:
   /192.168.7.209:50054[0](queued=0,recved=1,sent=0)
  
  Latency min/avg/max: 0/0/0
  Received: 1
  Sent: 0
  Connections: 1
  Outstanding: 0
  Zxid: 0x10000000d
  Mode: follower
  Node count: 4
  Connection closed by foreign host.
  ```

  

## 单机模式安装

* **1、修改 zoo.cfg**

  修改%ZK_HOME%/conf/zoo.cfg

  ```
  tickTime=2000
  initLimit=10
  syncLimit=5
  dataDir=/usr/local/zookeeper-3.4.5/zkData
  clientPort=2181
  server.1=127.0.0.1:2888:3888
  ```

* **2、在 dataDir 中创建 myid**

  即在/usr/local/zookeeper-3.4.5/zkData下创建 myid。

  * myid 文件中只有一个数字，server.1的 myid 文件内容是 "1"。

* **3、启动**

  在%ZK_HOME%/bin 目录下执行下面命令

  ```
  ./zkServer.sh start
  看到下面内容
  JMX enabled by default
  Using config: /usr/local/zookeeper-3.4.5/bin/../conf/zoo.cfg
  Starting zookeeper ... STARTED
  ```

* **4、验证**

  使用 telnet 和 stat 验证是否正常启动。

  ```
  telnet 127.0.0.1 2181
  Trying 127.0.0.1...
  Connected to localhost.
  Escape character is '^]'.
  stat
  Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
  Clients:
   /127.0.0.1:50073[0](queued=0,recved=1,sent=0)
  
  Latency min/avg/max: 0/0/0
  Received: 1
  Sent: 0
  Connections: 1
  Outstanding: 0
  Zxid: 0x1f
  Mode: standalone
  Node count: 5
  Connection closed by foreign host.
  ```

## 客户端脚本

* **连接脚本**

```
./zkCli.sh
```

连接指定的 zk 服务器

```
./zkCli.sh -server 192.168.7.30:2181
```

 * **创建**

   使用 **create** 命令，创建一个 zk 节点

   ```
   create [-s] [-e] path data acl
   ```

   -s 或 -e 分别指定节点特性：顺序或临时节点。默认情况下，创建的是持久节点。

   ```
   [zk: 192.168.7.30:2181(CONNECTED) 0] create /zk-book 123
   Created /zk-book
   ```

* **读取**

  使用 **ls** 命令和 **get** 命令

  使用 **ls**，读取 zk 指定节点下的所有子节点。

  ```
  ls path [watch]
  ```

  ```
  [zk: 192.168.7.30:2181(CONNECTED) 1] ls /
  [zk-book, zookeeper]
  ```

  使用 **get**，获取指定节点的数据内容和属性信息

  ```
  get path [watch]
  ```

  ```
  [zk: 192.168.7.30:2181(CONNECTED) 2] get /zk-book
  123
  cZxid = 0x300000004
  ctime = Sun Dec 22 21:31:03 CST 2019
  mZxid = 0x300000004
  mtime = Sun Dec 22 21:31:03 CST 2019
  pZxid = 0x300000004
  cversion = 0
  dataVersion = 0
  aclVersion = 0
  ephemeralOwner = 0x0
  dataLength = 3
  numChildren = 0
  ```



* **更新**

  使用 **set** 命令，更新指定节点的数据内容。

  ```
  set path data [version]
  ```

  ```
  [zk: 192.168.7.30:2181(CONNECTED) 3] set /zk-book 456
  cZxid = 0x300000004
  ctime = Sun Dec 22 21:31:03 CST 2019
  mZxid = 0x300000005
  mtime = Sun Dec 22 21:37:33 CST 2019
  pZxid = 0x300000004
  cversion = 0
  dataVersion = 1
  aclVersion = 0
  ephemeralOwner = 0x0
  dataLength = 3
  numChildren = 0
  ```

  

* 删除

  使用 **delete**命令，删除指定节点。

  ```
  delete path [version]
  ```

  ```
  [zk: 192.168.7.30:2181(CONNECTED) 4] delete /zk-book
  ```

  删除某个节点，该节点必须没有子节点。

  ```
  [zk: 192.168.7.30:2181(CONNECTED) 8] create /zk-book 123
  Created /zk-book
  [zk: 192.168.7.30:2181(CONNECTED) 9] create /zk-book/child 345
  Created /zk-book/child
  [zk: 192.168.7.30:2181(CONNECTED) 10] delete /zk-book
  Node not empty: /zk-book
  [zk: 192.168.7.30:2181(CONNECTED) 11] delete /zk-book/child
  [zk: 192.168.7.30:2181(CONNECTED) 12] delete /zk-book
  ```

## Java客户端 API 使用

* 创建一个最基本的 zk 会话实例

  ```java
  <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.4.9</version>
  </dependency>
  ```

  

  ```java
  package chap5;
  
  import java.io.IOException;
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooKeeper;
  
  public class Zk_Usage_Simple implements Watcher {
  
      private static CountDownLatch connectedSemaphore = new CountDownLatch(1);
  
      public static void main(String[] args) throws IOException {
          ZooKeeper zooKeeper = new ZooKeeper("192.168.7.30:2181", 5000, new Zk_Usage_Simple());
          System.out.println(zooKeeper.getState());
  
          try {
              connectedSemaphore.await();
          } catch (InterruptedException e) {
              System.out.println("zk session established");
          }
  
      }
  
      public void process(WatchedEvent event) {
          System.out.println("Received watched event: " + event);
  
          if (KeeperState.SyncConnected == event.getState()) {
              connectedSemaphore.countDown();
          }
  
      }
  }
  
  ```

* 同步 API 创建节点

  ```java
  package chap5;
  
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  
  public class Zk_Create_Sync_Usage implements Watcher {
      private static CountDownLatch connectedSemaphore = new CountDownLatch(1);
  
      public static void main(String[] args) throws Exception {
          ZooKeeper zooKeeper = new ZooKeeper("192.168.7.30:2181", 5000, new Zk_Create_Sync_Usage());
          connectedSemaphore.await();
  
          // 创建临时节点
          String path = zooKeeper.create("/zk-test-ephemeral-", "zk-test-ephemeral-".getBytes(), Ids.OPEN_ACL_UNSAFE,
                  CreateMode.EPHEMERAL);
  
          System.out.println("Success create zNode :" + path);
  
          // 创建临时节点并自动后缀加上数字
          String path2 = zooKeeper.create("/zk-test-ephemeral-", "zk-test-ephemeral-".getBytes(), Ids.OPEN_ACL_UNSAFE,
                  CreateMode.EPHEMERAL_SEQUENTIAL);
          System.out.println("Success create zNode :" + path2);
      }
  
      public void process(WatchedEvent event) {
          if (event.getState() == KeeperState.SyncConnected) {
              connectedSemaphore.countDown();
          }
      }
  }
  
  ```

  Success create zNode :/zk-test-ephemeral-
  Success create zNode :/zk-test-ephemeral-0000000007

* 异步 API 创建节点

  ```java
  package chap5;
  
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.AsyncCallback;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  
  public class Zk_Create_ASync_Usage implements Watcher {
      private static CountDownLatch connectedSemaphore = new CountDownLatch(1);
  
      public static void main(String[] args) throws Exception {
          ZooKeeper zooKeeper = new ZooKeeper("192.168.7.30:2181", 5000, new Zk_Create_ASync_Usage());
          connectedSemaphore.await();
  
          zooKeeper.create("/zk-test-ephemeral-", "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL,
                  new IStringCallback(), "I am context");
  
          zooKeeper.create("/zk-test-ephemeral-", "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL,
                  new IStringCallback(), "I am context");
  
          zooKeeper.create("/zk-test-ephemeral-", "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL,
                  new IStringCallback(), "I am context");
  
          Thread.sleep(Integer.MAX_VALUE);
  
      }
  
      public void process(WatchedEvent event) {
          if (event.getState() == KeeperState.SyncConnected) {
              connectedSemaphore.countDown();
          }
      }
  
  }
  
  
  class IStringCallback implements AsyncCallback.StringCallback {
      public void processResult(int rc, String path, Object ctx, String name) {
          System.out.println("Create path result : [" + rc + ", " + path + ", " + ctx + ", " + name);
      }
  }
  
  ```

  Create path result : [0, /zk-test-ephemeral-, I am context, /zk-test-ephemeral-
  Create path result : [-110, /zk-test-ephemeral-, I am context, null
  Create path result : [-110, /zk-test-ephemeral-, I am context, null

  和同步接口方法创建节点最大的区别在于，节点的创建过程（包括网络通信和服务端的节点创建过程）是异步的。并且，异步接口不会抛出异常，所有的异常都在回调中通过 Result Code 来体现。

* 删除节点

  ```java
  package chap5;
  
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  
  public class ZKDeleteSyncUsage implements Watcher {
      private static final CountDownLatch connectedSemaphore = new CountDownLatch(1);
  
      public static void main(String[] args) throws Exception {
          ZooKeeper zk = new ZooKeeper("192.168.7.30:2181", 5000, new ZKDeleteSyncUsage());
  
          connectedSemaphore.await();
  
          String path = "/zk-delete-node";
          zk.create(path, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
  
          // if the given version is -1, it matches any node's versions
          zk.delete(path, -1);
  
          Thread.sleep(Integer.MAX_VALUE);
      }
  
      public void process(WatchedEvent event) {
          if (event.getState() == KeeperState.SyncConnected) {
              connectedSemaphore.countDown();
          }
      }
  }
  
  ```

  在 zk 中，只允许删除子节点，如果一个节点有子节点，必须先删除掉所有子节点才能删除。

* 读取数据

  读取数据，包括子节点列表的获取和节点数据的获取。

  ```java
  package chap5;
  
  import java.util.List;
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.EventType;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  
  public class ZKGetChildrenSyncUsage implements Watcher {
  
      private static final CountDownLatch connectedSemaphore = new CountDownLatch(1);
  
      private static ZooKeeper zk = null;
  
      public static void main(String[] args) throws Exception {
          zk = new ZooKeeper("192.168.7.30:2181", 5000, new ZKGetChildrenSyncUsage());
          connectedSemaphore.await();
          String path = "/zk-book";
  
          zk.create(path, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
  
          zk.create(path + "/c1", "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
          List<String> childrenList = zk.getChildren(path, true);
          System.out.println("childrenList :" + childrenList);
  
          zk.create(path + "/c2", "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
  
          Thread.sleep(Integer.MAX_VALUE);
      }
  
      public void process(WatchedEvent event) {
          if (event.getState() == KeeperState.SyncConnected) {
              if (EventType.None == event.getType() && event.getPath() == null) {
                  connectedSemaphore.countDown();
              } else if (event.getType() == EventType.NodeChildrenChanged) {
                  try {
                      System.out.println("ReGet child:" + zk.getChildren(event.getPath(), true));
                  } catch (Exception e) {
  
                  }
              }
          }
      }
  }
  
  ```

  获取数据

  ```java
  package chap5;
  
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.EventType;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  import org.apache.zookeeper.data.Stat;
  
  public class ZKGetDataSyncUsage implements Watcher {
      private static final CountDownLatch connectedSemaphore = new CountDownLatch(1);
  
      private static ZooKeeper zk = null;
      private static Stat stat = new Stat();
  
      public static void main(String[] args) throws Exception {
          zk = new ZooKeeper("192.168.7.30:2181", 5000, new ZKGetDataSyncUsage());
          connectedSemaphore.await();
          String path = "/zk-book";
  
          zk.create(path, "123".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
          System.out.println(new String(zk.getData(path, true, stat)));
  
          System.out.println(stat.getCzxid() + " " + stat.getMzxid() + " " + stat.getVersion());
  
          zk.setData(path, "123".getBytes(), -1);
  
          Thread.sleep(Integer.MAX_VALUE);
      }
  
      public void process(WatchedEvent event) {
          if (event.getState() == KeeperState.SyncConnected) {
              if (EventType.None == event.getType() && event.getPath() == null) {
                  connectedSemaphore.countDown();
              } else if (event.getType() == EventType.NodeDataChanged) {
                  try {
                      System.out.println(new String(zk.getData(event.getPath(), true, stat)));
  
                      System.out.println(stat.getCzxid() + " " + stat.getMzxid() + " " + stat.getVersion());
                  } catch (Exception e) {
                      // EMPTY
                  }
              }
          }
      }
  }
  
  ```

  **节点的内容或是节点数据版本变化，都被看作节点变化**
  
* 更新数据

  ```java
  package chap5;
  
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.KeeperException;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.EventType;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  import org.apache.zookeeper.data.Stat;
  
  public class ZKSetDataSyncUsage implements Watcher {
  
      private static CountDownLatch connectedSemaphore = new CountDownLatch(1);
      private static ZooKeeper zk = null;
  
      public static void main(String[] args) throws Exception {
          zk = new ZooKeeper("192.168.7.30:2181", 5000, new ZKSetDataSyncUsage());
          connectedSemaphore.await();
          String path = "/zk-book";
  
          zk.create(path, "123".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
          zk.getData(path, true, null);
  
          Stat stat = zk.setData(path, "456".getBytes(), -1);
          System.out.println(
                  "Czxid = " + stat.getCzxid() + " Mzxid = " + stat.getMzxid() + " Version = " + stat.getVersion());
  
          Stat stat2 = zk.setData(path, "456".getBytes(), stat.getVersion());
          System.out.println(
                  "Czxid = " + stat.getCzxid() + " Mzxid = " + stat.getMzxid() + " Version = " + stat2.getVersion());
  
          try {
              zk.setData(path, "456".getBytes(), stat.getVersion());
  
          } catch (KeeperException e) {
              System.out.println("code = " + e.getCode() + "message = " + e.getMessage());
          }
  
          Thread.sleep(Integer.MAX_VALUE);
  
      }
  
      public void process(WatchedEvent event) {
          if (KeeperState.SyncConnected == event.getState()) {
              if (EventType.None == event.getType() && null == event.getPath()) {
                  connectedSemaphore.countDown();
              }
          }
      }
  }
  
  ```

  基于 version 参数，可以很好的控制 ZooKeeper 上节点数据的原子性操作。

* 检测节点是否存在

  ```java
  package chap5;
  
  import java.io.IOException;
  import java.util.concurrent.CountDownLatch;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.KeeperException;
  import org.apache.zookeeper.WatchedEvent;
  import org.apache.zookeeper.Watcher;
  import org.apache.zookeeper.Watcher.Event.EventType;
  import org.apache.zookeeper.Watcher.Event.KeeperState;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  
  public class ZKExistSyncUsage implements Watcher {
  
      private static final CountDownLatch connectedSemaphore = new CountDownLatch(1);
      private static ZooKeeper zk;
  
      public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
          String path = "/zk-book";
          zk = new ZooKeeper("192.168.7.30:2181", 5000, new ZKExistSyncUsage());
  
          connectedSemaphore.await();
  
          zk.exists(path, true);
  
          zk.create(path, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
  
          zk.setData(path, "123".getBytes(), -1);
  
          zk.create(path + "/c1", "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
  
          zk.delete(path + "/c1", -1);
  
          zk.delete(path, -1);
  
          Thread.sleep(Integer.MAX_VALUE);
  
      }
  
      public void process(WatchedEvent event) {
          try {
              if (event.getState() == KeeperState.SyncConnected) {
                  if (event.getType() == EventType.None && null == event.getPath()) {
                      connectedSemaphore.countDown();
                  } else if (event.getType() == EventType.NodeCreated) {
                      System.out.println("Node:" + event.getPath() + " Created");
                      zk.exists(event.getPath(), true);
                  } else if (event.getType() == EventType.NodeDeleted) {
                      System.out.println("Node:" + event.getPath() + " Deleted");
                      zk.exists(event.getPath(), true);
                  } else if (event.getType() == EventType.NodeDataChanged) {
                      System.out.println("Node:" + event.getPath() + " DataChanged");
                      zk.exists(event.getPath(), true);
                  }
              }
          } catch (Exception e) {
              System.out.println(e);
          }
  
      }
  }
  
  ```

  Node:/zk-book Created
  Node:/zk-book DataChanged
  Node:/zk-book Deleted

  * 无论节点是否存在，通过 exists 接口都可以注册 watcher。
  * exists 接口中注册的 watcher，能过对节点创建、节点删除、节点数据更新进行监听。
  * 对于指定节点的子节点的各种变化，都不会通知客户端。

* 权限控制

  ```java
  package chap5;
  
  import java.io.IOException;
  import org.apache.zookeeper.CreateMode;
  import org.apache.zookeeper.KeeperException;
  import org.apache.zookeeper.ZooDefs.Ids;
  import org.apache.zookeeper.ZooKeeper;
  
  public class ZKAuthUsage {
      public static void main(String[] args) throws IOException, KeeperException, InterruptedException {
          String path = "/zk-book-auth";
          String path2 = path + "/c1";
  
          ZooKeeper zk = new ZooKeeper("192.168.7.30:2181", 5000, new ZKCreateSyncUsage());
          zk.addAuthInfo("digest", "foo:foo".getBytes());
          zk.create(path, "123".getBytes(), Ids.CREATOR_ALL_ACL, CreateMode.PERSISTENT);
          zk.create(path2, "123".getBytes(), Ids.CREATOR_ALL_ACL, CreateMode.EPHEMERAL);
  
          ZooKeeper zk2 = new ZooKeeper("192.168.7.30:2181", 5000, new ZKCreateSyncUsage());
          zk2.addAuthInfo("digest", "foo:foo".getBytes());
          //        byte[] data = zk2.getData(path, false, null);
          //        System.out.println(new String(data));
          zk2.delete(path2, -1);
  
          ZooKeeper zk3 = new ZooKeeper("192.168.7.30:2181", 5000, new ZKCreateSyncUsage());
          zk3.delete(path, -1);
  
          Thread.sleep(Integer.MAX_VALUE);
      }
  }
  
  ```

  当客户端对一个节点添加权限信息后，其作用范围是节点，也就是说当我们对一个数据节点添加权限信息后，依然可以自由删除这个节点。但这个节点的子节点，就必须使用相应的权限信息才能删除。

## 典型使用场景

ZooKeeper 是一个高可用的分布式数据管理和协调框架，基于对 ZAB 算法的实现，该框架能够很好的保证分布式环境中数据的一致性。

* 数据发布/订阅

  数据发布/订阅系统，即所谓的配置中心。发布者将数据发布到 zk 的一个或一系列节点上，供订阅者进行数据订阅，实现配置信息的集中式管理和数据的动态更新。

  zk 采用推拉结合的方式，客户端向服务端注册自己关心的节点，一旦该节点的数据发生变更，那么服务端就向相应的服务端


