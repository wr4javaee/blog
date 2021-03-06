---
title:  "数据库学习笔记 - 数据库保护"
date: 2016-05-22 00:00:00
categories: Database
tags: 数据库
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---

## 什么是事务（Transaction）

事务是用户定义的数据操作系列，这些操作可作为一个完整的工作单元（数据库的逻辑工作单位）。
<!-- more -->

---



## 事务的特性（ACID）

* 原子性（Atomicity）事务中的操作要么都做，要么都不做。
* 一致性（Consistency）只有合法的数据可以被写入数据库，否则事务应该将其回滚到最初状态。
* 隔离性（Isolation）数据库中一个事务的执行不能被其他事务干扰。
* 持久性（Durability）事务一旦提交，其对数据库中数据的改变就是永久的。

---



## 并发控制

并发问题导致的异常情况归类如下：

#### 丢失更新（Lost Update）

两个事务都同时更新一行数据，第一个事务的提交结果被第二个事务的提交结果破坏。



![lost_update](/images/db/transaction/2016_05_26_lost_update.png)



#### 脏读（Dirty Reads）

一个事务读了某个失败事务运行过程中的数据。

![ldirty_reads](/images/db/transaction/2016_05_26_dirty_reads.png)



#### 不可重复读（Non-repeatable Reads）

一个事务对同一行数据重复读取两次，但是却得到了不同的结果。

![non_repeatable_reads](/images/db/transaction/2016_05_26_non_repeatable_reads.png)



#### 幻读（Phantom Reads）

幻读实际属于不可重复读的范畴。它指当事务T1按照一定条件读取某些数据，事务T2对其中部分记录做了删除或更新操作，当事务T1再次以相同条件读取数据时，发现少或多了某些记录。

---

## 事务的隔离级别

#### 未授权读取（Read Uncommitted）
也称读未提交，隔离级别最低，允许脏读。

#### 授权读取（Read Committed）
也称读已提交，允许不可重复读。

#### 可重复读（Repeatable Read）
允许出现幻读。

#### 串行化（Serializable）
最严格的事务隔离级别，要求所有事务都被串行执行。

![transaction_isolation_level](/images/db/transaction/2016_05_26_transaction_isolation_level.png)

| 隔离级别  | 脏读   | 可重复读 | 幻读   |
| ----- | ---- | ---- | ---- |
| 未授权读取 | 存在   | 不可以  | 存在   |
| 授权读取  | 不存在  | 不可以  | 存在   |
| 可重复读取 | 不存在  | 可以   | 存在   |
| 串行化   | 不存在  | 可以   | 不存在  |

# 分布式事务

## CAP定理
一个分布式系统不可能同时满足一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance），最多只能同时满足其中两项。

## BASE理论

# 一致性协议

协调者：统一调度所有分布式节点执行逻辑组件。
参与者：被协调者调度的分布式节点。

## 2PC二阶段提交协议（Two-Phase Commit）

## 3PC

# 锁

## 悲观锁（Pessimistic Concurrency Control， PCC）
又称排他锁，若事务T1使用悲观锁对数据进行处理，则处理完成前，其他事务均不能对数据进行更新操作。

## 乐观锁 （Optimistic Concurrency Control， OCC）
乐观锁事务控制分为三阶段，分别是：数据读取、写入校验、数据写入。

