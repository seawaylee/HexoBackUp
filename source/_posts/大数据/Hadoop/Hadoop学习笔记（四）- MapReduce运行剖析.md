---
title: Hadoop学习笔记（四）- MapReduce运行剖析 
date: 2017-06-12 20:00:26
tags: [Hadoop]
---


# 1 MapReduce中的Shuffle机制

## 1.1 Shuffle概述

MR中，Map阶段处理的数据如何传递给Reduce阶段是MR框架中最关键的一个流程--**Shuffle**。
Shuffle的核心机制：**数据分区、排序、缓存**
具体来说就是将MapTask输出的处理结果分发给ReduceTask，并在分发的过程中对数据**按key进行了分区和排序**。

Shuffle缓存流程
![](shuffle缓存流程.png)

<!--more-->

## 1.2 Shuffle过程

Shuffle是MR处理流程中的一个过程，它的每一个处理步骤是分散在各个MapTask和ReduceTask节点上完成的，整体来看，分为3个操作：

1. 分区partition
2. Sort根据key排序
3. Combiner进行局部value的合并

**详细流程**

1. MapTask收集map方法输出的kv对，放到内存缓冲区中
2. 从内从缓冲区中不断溢出到本地磁盘文件，可能会溢出多个文件
3. 多个溢出文件会被合并成大的溢出文件
4. 在溢出及合并的过程中，都要调用partitioner进行分组和针对key进行排序（快速排序）
5. ReduceTask根据自己的分区号，去各个Maptask机器上取响应的结果分区数据
6. ReduceTask会取到同一个分区的、来自不同MapTask的结果文件，并将这些文件再进行合并（归并排序）
7. 合并成大文件后，shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中逐一取出K-VGroup，调用用户自定义的reduce方法）

Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，**缓冲区越大，磁盘的IO次数越少，执行速度就越快**。缓冲区的大小可以通过参数io.sort.mb调整，默认为100M。

![](mapreduce的shuffle原理.png)


 

# 2 MapReduce与Yarn

## 2.1 Yarn概述

Yarn是一个资源调度平台，负责为运算程序提供服务器的运算资源，相当于一个分布式的OS平台，而MapReduce等运算程序则相当于运行于OS之上的应用程序。

1. yarn并**不清楚**用户提交程序的运行机制
2. yarn只提供**运算资源的调度**（用户向yarn申请资源，yarn就负责分配资源）
3. yarn中的主管角色叫**ResourceManager**
4. yarn中具体提供运算资源的角色叫**NodeManager**
5. yarn与运行的用户程序**完全解耦**，这就意味着yarn上可以运行各种类型的分布式运算程序，例如**MR、Storm、Spark、Tez....**
6. 所以Spark、Storm等运算框架都可以整合在yarn上运行，只要他们各自的框架中有**符合yarn规范的资源请求机制**即可。
7. yarn就成为一个**通用的资源调度平台**，从此，企业中以前存在在的各种运算集群都可以整合在一个物理集群上，提高资源利用率，方便数据共享。


![](MR在Yarn的调度过程.png)

# 3 MR运算全流程图

![](mr全流程原理全剖析.png)