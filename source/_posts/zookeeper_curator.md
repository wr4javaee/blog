---
title:  "Zookeeper Curator的使用"
date: 2016-07-27 00:00:00
categories: Zookeeper
tags: Zookeeper Curator
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---

# 前言
Curator是ZooKeeper的开源客户端。
官网：http://curator.apache.org/
API文档：http://curator.apache.org/apidocs/index.html
<!-- more -->

在工程中需要引入curator依赖

```xml
	<dependency>
		<groupId>org.apache.curator</groupId>
		<artifactId>curator-framework</artifactId>
		<version>${curator.version}</version>
	</dependency>
```


截止目前文章中使用的版本为3.2.0
---

## 创建Session

###  使用CuratorFrameworkFactory创建Session

可以通过newClient方法建立ZooKeeper会话，但这里推荐使用其自带的Builder模式，下面是newClient的官方API，Builder模式的case见下文。

- #### newClient

  ```
  public static CuratorFramework newClient(String connectString,
                                           RetryPolicy retryPolicy)
  ```

  Create a new client with default session timeout and default connection timeout

  - Parameters:

    `connectString` - list of servers to connect to

    `retryPolicy` - retry policy to use

  - Returns:

    client


- #### newClient

  ```
  public static CuratorFramework newClient(String connectString,
                                           int sessionTimeoutMs,
                                           int connectionTimeoutMs,
                                           RetryPolicy retryPolicy)
  ```

  Create a new client

  - Parameters:

    `connectString` - list of servers to connect to

    `sessionTimeoutMs` - session timeout

    `connectionTimeoutMs` - connection timeout

    `retryPolicy` - retry policy to use

  - Returns:

    client

    ​


RetryPolicy是Curator提供的创建会话的策略接口。

```
/**
 * Abstracts the policy to use when retrying connections
 */
public interface RetryPolicy
{
    /**
     * Called when an operation has failed for some reason. This method should return
     * true to make another attempt.
     *
     *
     * @param retryCount the number of times retried so far (0 the first time)
     * @param elapsedTimeMs the elapsed time in ms since the operation was attempted
     * @param sleeper use this to sleep - DO NOT call Thread.sleep
     * @return true/false
     */
    public boolean      allowRetry(int retryCount, long elapsedTimeMs, RetrySleeper sleeper);
}
```

官方提供的RetryPolicy的继承关系如下，我们也可通过继承RetryPolicy来实现自己的策略类。

![retrypolicy_extends](images/curator/retrypolicy_extends.png)

这里，我们使用ExponentialBackoffRetry这个重试策略来建立ZooKeeperSession。

### case

```java
package me.wangran.zookeeper.demo.curator;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;

/**
 * Curator创建SessionDemo
 * @author Wang Ran
 */
public class CuratorCreateSessionDemo {

	public static void main(String[] args) {
		createSessionDemo1();
	}
	
	/**
	 * 使用CuratorFrameworkFactory创建Session
	 */
	public static void createSessionDemo1() {
		String connectString = "192.168.128.75:2181";
		int baseSleepTimeMs = 1000; // 初始sleep时间
		int maxRetries = 100; // 最大重试次数
		int maxSleepMs = 25000; // 最大sleep时间
		// ExponentialBackoffRetry : Retry policy that retries a set number of 
		// times with increasing sleep time between retries
		RetryPolicy retryPolicy = new ExponentialBackoffRetry(baseSleepTimeMs, maxRetries, maxSleepMs);
		int sessionTimeoutMs = 10000;
		int connectionTimeoutMs = 10000;
		CuratorFramework cff = CuratorFrameworkFactory.builder()
				.connectString(connectString)
				.retryPolicy(retryPolicy)
				.sessionTimeoutMs(sessionTimeoutMs)
				.connectionTimeoutMs(connectionTimeoutMs)
				.build();
		cff.start();
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("ZooKeeper current state is " + cff.getState());
	}
}
```

当我运行该代码后，控制台报错如下：

```
Exception in thread "main" java.lang.NoSuchMethodError: org.apache.zookeeper.server.quorum.flexible.QuorumMaj.<init>(Ljava/util/Map;)V
	at org.apache.curator.framework.imps.EnsembleTracker.<init>(EnsembleTracker.java:57)
	at org.apache.curator.framework.imps.CuratorFrameworkImpl.<init>(CuratorFrameworkImpl.java:158)
	at org.apache.curator.framework.CuratorFrameworkFactory$Builder.build(CuratorFrameworkFactory.java:156)
	at me.wangran.zookeeper.demo.curator.CuratorCreateSessionDemo.createSessionDemo1(CuratorCreateSessionDemo.java:36)
	at me.wangran.zookeeper.demo.curator.CuratorCreateSessionDemo.main(CuratorCreateSessionDemo.java:15)
```

报错的原因如下：

> Curator 2.x.x - compatible with both ZooKeeper 3.4.x and ZooKeeper 3.5.x
> Curator 3.x.x - compatible only with ZooKeeper 3.5.x and includes support for new features such as dynamic reconfiguration, etc.

在依赖中，我引入的curator与zookeeper版本不兼容导致

>   <properties>
>   	<zookeeper.version>3.4.6</zookeeper.version>
>   	<curator.version>3.2.0</curator.version>
>   </properties>

将zookeeper版本改为3.5.2-alpha后ZooKeeper会话创建成功，控制台输出如下：

```
ZooKeeper current state is STARTED
```

这里特别说明一下，这个zookeeper版本指的是maven依赖的客户端版本，并不是zookeeper服务本身的版本。