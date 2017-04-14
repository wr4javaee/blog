---
title:  "Zookeeper Java API的使用"
date: 2016-07-26 00:00:00
categories: Zookeeper
tags: Zookeeper
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---

## 前言

Zookeeper官方API文档 http://zookeeper.apache.org/doc/r3.4.6/api/index.html

<!-- more -->

## 通过API建立ZooKeeper Session

ZooKeeper构造方法官方文档摘录如下

> | Constructor and Description              |
> | ---------------------------------------- |
> | `ZooKeeper(String connectString,         int sessionTimeout,         Watcher watcher)`To create a ZooKeeper client object, the application needs to pass a connection string containing a comma separated list of host:port pairs, each corresponding to a ZooKeeper server. |
> | `ZooKeeper(String connectString,         int sessionTimeout,         Watcher watcher,         boolean canBeReadOnly)`To create a ZooKeeper client object, the application needs to pass a connection string containing a comma separated list of host:port pairs, each corresponding to a ZooKeeper server. |
> | `ZooKeeper(String connectString,         int sessionTimeout,         Watcher watcher,         long sessionId,         byte[] sessionPasswd)`To create a ZooKeeper client object, the application needs to pass a connection string containing a comma separated list of host:port pairs, each corresponding to a ZooKeeper server. |
> | `ZooKeeper(String connectString,         int sessionTimeout,         Watcher watcher,         long sessionId,         byte[] sessionPasswd,         boolean canBeReadOnly)`To create a ZooKeeper client object, the application needs to pass a connection string containing a comma separated list of host:port pairs, each corresponding to a ZooKeeper server. |



Demo github地址：

```java
package me.wangran.zookeeper.demo.api;

import java.util.concurrent.CountDownLatch;

import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.Watcher.Event.KeeperState;
import org.apache.zookeeper.ZooKeeper;

/**
 * Zookeeper 创建Session
 * @author Wang Ran
 */
public class ZooKeeperSessionDemo implements Watcher {
	private static CountDownLatch countDownLatch = new CountDownLatch(1);
	
	public static void main(String[] args) throws Exception {
		demo3();
	}
	
	/**
	 * 创建最基本的会话
	 */
	public static void demo1() throws Exception {
		String zookeeperAddress = "192.168.128.75:2181"; // Zookeeper地址，多个Zookeeper地址可以英文逗号分隔，地址可指定节点
		int sessionTimeout = 10000; // session超时时间，单位毫秒
		Watcher watcher = new ZooKeeperSessionDemo(); // 事件通知处理器， 创建会话时可为null
		ZooKeeper zookeeper = new ZooKeeper(zookeeperAddress, sessionTimeout, watcher);
		System.out.println("zookeeper connecting, state is " + zookeeper.getState());
		countDownLatch.await();
		System.out.println("zookeeper connected, state is " + zookeeper.getState());
	}
	
	/**
	 * 创建指定read only模式的会话
	 */
	public static void demo2() throws Exception {
		String zookeeperAddress = "192.168.128.75:2181/test_root"; // Zookeeper地址，多个Zookeeper地址可以英文逗号分隔，地址可指定节点
		int sessionTimeout = 10000; // session超时时间，单位毫秒
		Watcher watcher = new ZooKeeperSessionDemo(); // 事件通知处理器， 创建会话时可为null
		boolean canBeReadOnly = true; // 是否支持read only模式，默认为false，若一个机器和Zookeeper Cluster过半机器失去网络连接，则不处理所有读写请求。若未true，则该情况可提供读请求。
		ZooKeeper zookeeper = new ZooKeeper(zookeeperAddress, sessionTimeout, watcher, canBeReadOnly);
		System.out.println("zookeeper connecting, state is " + zookeeper.getState());
		countDownLatch.await();
		System.out.println("zookeeper connected, state is " + zookeeper.getState());
	}
	
	/**
	 * 恢复会话
	 */
	public static void demo3()  throws Exception {
		String zookeeperAddress = "192.168.128.75:2181/test_root"; // Zookeeper地址，多个Zookeeper地址可以英文逗号分隔，地址可指定节点
		int sessionTimeout = 10000; // session超时时间，单位毫秒
		Watcher watcher = new ZooKeeperSessionDemo(); // 事件通知处理器， 创建会话时可为null
		ZooKeeper zookeeper = new ZooKeeper(zookeeperAddress, sessionTimeout, watcher);
		System.out.println("zookeeper connecting, state is " + zookeeper.getState());
		countDownLatch.await();
		System.out.println("zookeeper connected, state is " + zookeeper.getState());
		long sessionId = zookeeper.getSessionId(); // 用于恢复会话
		byte[] sessionPwd = zookeeper.getSessionPasswd(); // 用于恢复会话
		countDownLatch = new CountDownLatch(1);
		zookeeper = new ZooKeeper(zookeeperAddress, sessionTimeout, watcher, sessionId, sessionPwd);
		System.out.println("zookeeper reconnecting, state is " + zookeeper.getState());
		countDownLatch.await();
		System.out.println("zookeeper reconnected, state is " + zookeeper.getState());
	}
	
	@Override
	public void process(WatchedEvent watchedEvent) {
		System.out.println("ZooKeeperWatcherDemo process, watchedEvent is " + watchedEvent);
		if(KeeperState.SyncConnected == watchedEvent.getState()) {
			countDownLatch.countDown();
		}
	}
}
```

---



## 通过API创建节点

### 说明

创建节点API地址：http://zookeeper.apache.org/doc/r3.4.6/api/index.html

这里，我们主要用到两个方法，分别是

| 返回值      | 方法                                       |
| -------- | ---------------------------------------- |
| `String` | `create(String path,      byte[] data,      List acl,      CreateMode createMode)`Create a node with the given path. |
| `void`   | `create(String path,      byte[] data,      List acl,      CreateMode createMode,      AsyncCallback.StringCallback cb,      Object ctx)`The asynchronous version of create. |



* path 指定创建的节点路径
* data 节点的内容
* acl 创建节点策略

```Java
        /**
         * This Id represents anyone.
         */
        public final Id ANYONE_ID_UNSAFE = new Id("world", "anyone");

        /**
         * This Id is only usable to set ACLs. It will get substituted with the
         * Id's the client authenticated with.
         */
        public final Id AUTH_IDS = new Id("auth", "");

        /**
         * This is a completely open ACL .
         */
        public final ArrayList<ACL> OPEN_ACL_UNSAFE = new ArrayList<ACL>(
                Collections.singletonList(new ACL(Perms.ALL, ANYONE_ID_UNSAFE)));

        /**
         * This ACL gives the creators authentication id's all permissions.
         */
        public final ArrayList<ACL> CREATOR_ALL_ACL = new ArrayList<ACL>(
                Collections.singletonList(new ACL(Perms.ALL, AUTH_IDS)));

        /**
         * This ACL gives the world the ability to read.
         */
        public final ArrayList<ACL> READ_ACL_UNSAFE = new ArrayList<ACL>(
                Collections
                        .singletonList(new ACL(Perms.READ, ANYONE_ID_UNSAFE)));
```

* cb 异步回调函数，当节点创建后ZooKeeper客户端自动调用本方法。需要实现StringCallback接口，重写processResult方法
* ctx 回调方法执行时使用的对象，通常放置上线文Context信息


* CreateMode节点类型枚举如下

```Java
    /**
     * The znode will not be automatically deleted upon client's disconnect
     * 持久节点
     */
    PERSISTENT (0, false, false, false),
    /**
    * The znode will not be automatically deleted upon client's disconnect,
    * and its name will be appended with a monotonically increasing number.
    * 持久顺序节点
    */
    PERSISTENT_SEQUENTIAL (2, false, true, false),
    /**
     * The znode will be deleted upon the client's disconnect.
     * 临时节点
     */
    EPHEMERAL (1, true, false, false),
    /**
     * The znode will be deleted upon the client's disconnect, and its name
     * will be appended with a monotonically increasing number.
     * 临时顺序节点
     */
    EPHEMERAL_SEQUENTIAL (3, true, true, false),
    /**
     * The znode will be a container node. Container
     * nodes are special purpose nodes useful for recipes such as leader, lock,
     * etc. When the last child of a container is deleted, the container becomes
     * a candidate to be deleted by the server at some point in the future.
     * Given this property, you should be prepared to get
     * {@link org.apache.zookeeper.KeeperException.NoNodeException}
     * when creating children inside of this container node.
     */
    CONTAINER (4, false, false, true);
```

### Case

* case 1 根节点下创建持久节点/node_pokemon

```Java
package me.wangran.zookeeper.demo.api;

import java.util.concurrent.CountDownLatch;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.Watcher.Event.KeeperState;
import org.apache.zookeeper.ZooDefs.Ids;

/**
 * ZooKeeper创建节点Demo
 * @author Wang Ran
 */
public class ZooKeeperCreateNodeDemo  implements Watcher {
	private static CountDownLatch countDownLatch = new CountDownLatch(1);
	
	public static void main(String[] args) throws Exception {
		new ZooKeeperCreateNodeDemo().demo1();
	}
	
	/**
	 * 创建最基本的会话
	 */
	public void demo1() throws Exception {
		String zookeeperAddress = "192.168.128.75:2181"; // Zookeeper地址，多个Zookeeper地址可以英文逗号分隔，地址可指定节点
		int sessionTimeout = 10000; // session超时时间，单位毫秒
		Watcher watcher = new ZooKeeperCreateNodeDemo(); // 事件通知处理器， 创建会话时可为null
		ZooKeeper zookeeper = new ZooKeeper(zookeeperAddress, sessionTimeout, watcher);
		System.out.println("zookeeper connecting, state is " + zookeeper.getState());
		countDownLatch.await();
		System.out.println("zookeeper connected, state is " + zookeeper.getState());
		
		String newNodePath = "/node_pokemon";
		byte[] newNodeData = "go".getBytes();
		CreateMode newNodeMode = CreateMode.PERSISTENT;
		zookeeper.create(newNodePath, newNodeData, Ids.OPEN_ACL_UNSAFE, newNodeMode);
		System.out.println("zookeeper node created");
	}
	
	@Override
	public void process(WatchedEvent watchedEvent) {
		System.out.println("ZooKeeperWatcherDemo process, watchedEvent is " + watchedEvent);
		if(KeeperState.SyncConnected == watchedEvent.getState()) {
			countDownLatch.countDown();
		}
	}
	
}
```

控制台输出：

> zookeeper connecting, state is CONNECTING
> ZooKeeperWatcherDemo process, watchedEvent is WatchedEvent state:SyncConnected type:None path:null
> zookeeper connected, state is CONNECTED
> zookeeper node created



* case2 创建节点后异步触发回调函数

```Java
package me.wangran.zookeeper.demo.api;

import java.util.concurrent.CountDownLatch;

import org.apache.zookeeper.AsyncCallback;
import org.apache.zookeeper.AsyncCallback.StringCallback;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.Watcher.Event.KeeperState;
import org.apache.zookeeper.ZooDefs.Ids;

/**
 * ZooKeeper创建节点Demo
 * @author Wang Ran
 */
public class ZooKeeperCreateNodeDemo  implements Watcher {
	private static CountDownLatch countDownLatch = new CountDownLatch(1);
	
	public static void main(String[] args) throws Exception {
		new ZooKeeperCreateNodeDemo().demo2();
	}
	
	/**
	 * 创建持久节点
	 */
	public void demo1() throws Exception {
		String zookeeperAddress = "192.168.128.75:2181"; // Zookeeper地址，多个Zookeeper地址可以英文逗号分隔，地址可指定节点
		int sessionTimeout = 10000; // session超时时间，单位毫秒
		Watcher watcher = new ZooKeeperCreateNodeDemo(); // 事件通知处理器， 创建会话时可为null
		ZooKeeper zookeeper = new ZooKeeper(zookeeperAddress, sessionTimeout, watcher);
		System.out.println("zookeeper connecting, state is " + zookeeper.getState());
		countDownLatch.await();
		System.out.println("zookeeper connected, state is " + zookeeper.getState());
		
		String newNodePath = "/node_pokemon";
		byte[] newNodeData = "go".getBytes();
		CreateMode newNodeMode = CreateMode.PERSISTENT;
		zookeeper.create(newNodePath, newNodeData, Ids.OPEN_ACL_UNSAFE, newNodeMode);
		System.out.println("zookeeper node created");
	}
	
	/**
	 * 创建节点后异步回调业务函数
	 */
	public void demo2() throws Exception {
		String zookeeperAddress = "192.168.128.75:2181/node_pokemon"; // Zookeeper地址，多个Zookeeper地址可以英文逗号分隔，地址可指定节点
		int sessionTimeout = 10000; // session超时时间，单位毫秒
		Watcher watcher = new ZooKeeperCreateNodeDemo(); // 事件通知处理器， 创建会话时可为null
		ZooKeeper zookeeper = new ZooKeeper(zookeeperAddress, sessionTimeout, watcher);
		System.out.println("zookeeper connecting, state is " + zookeeper.getState());
		countDownLatch.await();
		System.out.println("zookeeper connected, state is " + zookeeper.getState());
		
		String newNodePath = "/node_kinosaki";
		byte[] newNodeData = "osen".getBytes();
		CreateMode newNodeMode = CreateMode.PERSISTENT_SEQUENTIAL;
		Object ctx = "ContextDemo";
		zookeeper.create(newNodePath, newNodeData, Ids.OPEN_ACL_UNSAFE, newNodeMode, new StringCallbackDemo(), ctx);
		System.out.println("zookeeper node created");
		Thread.sleep(10);
	}
	
	/**
	 * 异步回调函数
	 * @author Wang Ran
	 */
	class StringCallbackDemo implements AsyncCallback.StringCallback {
		@Override
		public void processResult(int rc, String path, Object ctx, String name) {
			System.out.println("StringCallbackDemo.processResult executed!, resultCode is " + rc
					+ ", path is" + path // 调用接口时传入的Path
					+ ", context is " + ctx.toString() // 调用接口时传入的Context
					+ ", node name is " + name); // 实际在服务器中创建的节点名称
		}
	}
	
	@Override
	public void process(WatchedEvent watchedEvent) {
		System.out.println("ZooKeeperWatcherDemo process, watchedEvent is " + watchedEvent);
		if(KeeperState.SyncConnected == watchedEvent.getState()) {
			countDownLatch.countDown();
		}
	}
}
```

控制台输出：

> zookeeper connecting, state is CONNECTING
> ZooKeeperWatcherDemo process, watchedEvent is WatchedEvent state:SyncConnected type:None path:null
> zookeeper connected, state is CONNECTED
> zookeeper node created
> StringCallbackDemo.processResult executed!, resultCode is 0, path is/node_kinosaki, context is ContextDemo, node name is /node_kinosaki0000000005

---


## 通过API读取节点

### API说明：

* ### getChildren查询指定节点的所有子节点

* ### getData查询指定节点的数据内容

| 返回值    | 方法                                       |
| ------ | ---------------------------------------- |
| `List` | `  getChildren  (String path, boolean watch)`Return the list of the children of the node of the given path. |
| `void` | `  getChildren  (String path, boolean watch, AsyncCallback.Children2Callback cb, Object ctx)`The asynchronous version of getChildren. |
| `void` | `  getChildren  (String path, boolean watch, AsyncCallback.ChildrenCallback cb, Object ctx)`The asynchronous version of getChildren. |
| `List` | `  getChildren  (String path, boolean watch, Stat stat)`For the given znode path return the stat and children list. |
| `List` | `  getChildren  (String path, Watcher watcher)`Return the list of the children of the node of the given path. |
| `void` | `  getChildren  (String path, Watcher watcher, AsyncCallback.Children2Callback cb, Object ctx)`The asynchronous version of getChildren. |
| `void` | `  getChildren  (String path, Watcher watcher, AsyncCallback.ChildrenCallback cb, Object ctx)`The asynchronous version of getChildren. |
| `List` | `  getChildren  (String path, Watcher watcher, Stat stat)`For the given znode path return the stat and children list. |


| 返回值      | 方法                                       |
| -------- | ---------------------------------------- |
| `void`   | `getData(String path,       boolean watch,       AsyncCallback.DataCallback cb,       Object ctx)`The asynchronous version of getData. |
| `byte[]` | `getData(String path,       boolean watch,       Stat stat)`Return the data and the stat of the node of the given path. |
| `void`   | `getData(String path,       Watcher watcher,       AsyncCallback.DataCallback cb,       Object ctx)`The asynchronous version of getData. |
| `byte[]` | `getData(String path,       Watcher watcher,       Stat stat)`Return the data and the stat of the node of the given path. |

* ### 参数说明

| 参数      | 说明                                       |
| ------- | ---------------------------------------- |
| path    | 想要查询的节点路径。                               |
| watch   | false为不需要注册watcher，true为使用默认watcher。     |
| watcher | 可为null。指定watcher用于订阅子节点列表变化通知。当子节点被添加或删除时会向客户端发送通知（通知内容不包含节点列表，客户端需重新查询以获得节点列表信息）。 |
| cb      | 异步回调函数                                   |
| ctx     | 上下文对象                                    |
| stat    | 指定数据节点的状态信息，包含cZxid（节点创建的事务id）、mZxid（节点最后一次更新的事务id）、dataLength（节点数据内容长度），传入变量的值会在方法执行过程中被服务器响应的值替换。 |



## 通过API更新节点

### API说明

| 返回值    | 方法                                       |
| ------ | ---------------------------------------- |
| `Stat` | `  setData  (String path,       byte[] data,       int version)`Set the data for the node of the given path if such a node exists and the given version matches the version of the node (if the given version is -1, it matches any node's versions).说明：该方法为同步更新方法 |
| `void` | `  setData  (String path,       byte[] data,       int version,       AsyncCallback.StatCallback cb,       Object ctx)`The asynchronous version of setData.说明：该方法为异步更新方法 |

### 参数说明

| 参数      | 说明         |
| ------- | ---------- |
| path    | 想要更新的节点路径。 |
| data[ ] | 更新的内容      |
| version | 指定更新的数据版本  |
| cb      | 异步回调函数     |
| ctx     | 上下文对象      |




## 通过API删除节点

### API说明

| 返回值    | 方法                                       |
| ------ | ---------------------------------------- |
| `void` | `  delete  (String path,      int version)`Delete the node with the given path. |
| `void` | `  delete  (String path,      int version,      AsyncCallback.VoidCallback cb,      Object ctx)`The asynchronous version of delete. |

---



## 通过API判断节点是否存在

### API说明

| 返回值    | 方法                                       |
| ------ | ---------------------------------------- |
| `Stat` | `  exists  (String path,      boolean watch)`Return the stat of the node of the given path. |
| `void` | `  exists  (String path,      boolean watch,      AsyncCallback.StatCallback cb,      Object ctx)`The asynchronous version of exists. |
| `Stat` | `  exists  (String path,      Watcher watcher)`Return the stat of the node of the given path. |
| `void` | `  exists  (String path,      Watcher watcher,      AsyncCallback.StatCallback cb,      Object ctx)`The asynchronous version of exists. |

---



## 权限控制

### API说明

| 返回值    | 方法                                       |
| ------ | ---------------------------------------- |
| `List` | `  getACL  (String path,      Stat stat)`Return the ACL and stat of the node of the given path. |
| `void` | `  getACL  (String path,      Stat stat,      AsyncCallback.ACLCallback cb,      Object ctx)`The asynchronous version of getACL. |
| `Stat` | `  setACL  (String path,      List acl,      int version)`Set the ACL for the node of the given path if such a node exists and the given version matches the version of the node. |
| `void` | `  setACL  (String path,      List acl,      int version,      AsyncCallback.StatCallback cb,      Object ctx)`The asynchronous version of setACL. |
