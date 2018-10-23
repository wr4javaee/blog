---
title:  "Zookeeper学习笔记二"
date: 2016-07-18 00:00:00
categories: Zookeeper
tags: Zookeeper
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---

# 基本概念

### 集群角色

ZooKeeper没有沿用传统集群模式（主备模式，Master/Slave），而是采用了Leader、Follower、Observer三种角色。

* Leader 选举而来，为client提供读/写服务。
* Follower 为client提供读服务，可参与选举。
* Observer 为client提供读服务，不参与Leader选举，不参与写操作的“过半写成功”策略，通常用于提升集群读性能。
<!-- more -->
---




### 会话（Session）

---

### 数据节点（Znode）

指数据模型（Znode Tree）中的数据单元

---

### 版本

---

### 事件监听器（Watcher）

---

### ACL（Access Control Lists）

ZooKeeper定义了5种权限：

* CREATE 创建子节点的权限
* READ 获取节点数据和节点列表的权限
* WRITE 更新节点数据的权限
* DELETE 删除子节点的权限
* ADMIN 设置节点ACL的权限

---

## ZAB协议

ZooKeeper原子消息广播协议（Zookeeper Atomic Broadcast）。
ZooKeeper并未采用Paxos，而是采用ZAB作为数据一致性的核心算法。

### ZAB协议两种基本模式
#### 崩溃恢复
  服务器启动、Leader服务器挂掉时进入该模式，选举产生新Leader服务器。
  当集群中有过半服务器（包含Leader）与Leader服务器完成状态同步（即数据同步）后，退出该模式。

##### 基本特性
* ZAB协议需要确保已在Leader提交的事务最终被所有服务器提交。
* ZAB协议需要确保丢弃那些只在Leader服务器上被提出的事务。

##### Leader选举
算法思路：确保已被Leader提交的事务Proposal，同时丢弃已被掉过的事务Proposal。
即保证选举出的Leader服务器拥有集群中ZXID最大的事务Proposal。

##### 数据同步
Leader服务器通过确认事务日志中所有Proposal是否已被集群中过半的服务器提交，来判断是否完成数据同步。





#### 消息广播

##### ZAB协议核心
定义了可能改变ZooKeeper服务器数据状态的数据请求处理方式：
![zab_message_broadcast](/images/zookeeper/zab/2016_07_18_zab_message_broadcast.png)

* Leader服务器收到事务请求后，生成事务Proposal，并为其分配事务ID（全局单调递增的ID，ZXID），放入队列中。
* Follower服务器接收到Proposal后，以事务日志形式写入到本地磁盘，写入成功后Respone Leader服务器“Ack”。
* 当Leader服务器接收超过半数Follower服务器Ack响应，广播Commit消息发送给所有Follower服务器，Leader自身与收到Commit消息的Follower服务器会完成事务提交。

基于TCP协议（FIFO）保证消息接收与发送的顺序性。



