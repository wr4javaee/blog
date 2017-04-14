---
title:  "Zookeeper学习笔记一之Paxos算法"
date: 2016-07-10 00:00:00
categories: Zookeeper
tags: Zookeeper Paxos
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---

## 背景

1990年提出的一种基于消息传递且具有高度容错性的一致性算法。
<!-- more -->



## 算法描述

假设有一组可以提出提案的进程集合，对于一个一致性算法来说需保证：

* 提案只有一个会被选定
* 若没有提案提出，则没有被选定的提案
* 提案若被选定，进程可以获取提案信息



分布式系统的【三态】：成功、失败、超时。
