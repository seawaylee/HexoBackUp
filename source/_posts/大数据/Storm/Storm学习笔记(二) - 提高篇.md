---
title: Storm学习笔记(二) - 提高篇  
date: 2017-12-17 15:33:20
tags: [大数据,Storm]
---

**在基础篇中学习了以下内容**

1. Storm概述
2. Storm核心组件 及 并发机制
3. Storm集群的部署和测试
4. Storm的WordCount程序

**本文主要学习以下内容**

1. Storm框架通信机制（Worker内部通信有外部通信）
2. Storm组件本地目录树
3. Storm ZooKeeper 目录树
4. Storm任务提交的过程

<!--more-->

# 1 Storm通信机制

- Worker之间通信: 通常需要通过网络跨节点通信，0.9以后的Storm使用Netty作为进程间通信的消息框架。
- Worker进程内部通信: 不同Worker的Thread通信通常使用Disruptor来完成。
- Topology之间的通信: Storm不负责，需要自己实现，例如使用Kafka。

## 1.1 Worker进程间通信

### 1.1.1 Worker进程间通信概述


- 对于worker进程来说，为了管理流入和传出的消息，**每个worker进程**有一个独立的**接收线程**(对配置的TCP端口supervisor.slots.ports进行监听)
- 对应Worker接收线程，**每个worker存在一个独立的发送线程**，它负责从worker的transfer-queue中读取消息，并通过网络发送给其他worker
* 每个**executor**有自己的**incoming-queue和outgoing-queue**。
* **Worker接收线程**将收到的消息通过**task编号**传递给对应的**executor(一个或多个)的incoming-queues**;
* 每个executor有单独的线程分别来处理spout/bolt的业务逻辑，业务逻辑输出的中间数据会存放在**outgoing-queue**中，当executor的outgoing-queue中的**tuple达到一定的阀值**，executor的发送线程将批量获取outgoing-queue中的tuple,并发送到**transfer-queue**中。
* 每个worker进程控制一个或多个**executor线程**，用户可在代码中进行配置。其实就是我们在代码中设置的**并发度**个数。

![](15134826177130.jpg)



### 1.1.2 Worker进程间通信分析

* Worker接受线程通过网络接受数据，并**根据Tuple中包含的taskId，匹配到对应的executor**；然后根据executor找到对应的incoming-queue，**将数据存发送到incoming-queue队列中**。
* 业务逻辑执行线程**消费incoming-queue的数据**，通过调用Bolt的**execute(xxxx)**方法，将Tuple作为参数传输给用户自定义的方法
* 业务逻辑执行完毕之后，**将计算的中间数据发送给outgoing-queue队列**，当outgoing-queue中的tuple达到一定的**阀值**，executor的**发送线程**将批量获取outgoing-queue中的tuple,并发送到Worker的transfer-queue中
* **Worker发送线程消费transfer-queue中数据**，计算Tuple的目的地，连接不同的**node+port**将数据通过网络传输的方式传送给另一个的Worker。
* 另一个worker执行以上步骤1的操作。


![](15134877769104.jpg)


### 1.1.3 Worker进程间通信技术（Netty、ZeroMQ）

**Netty**

Netty是一个NIO client-server(客户端服务器)框架，使用Netty可以快速开发网络应用，例如服务器和客户端协议。Netty提供了一种新的方式来使开发网络应用程序，这种新的方式使得它很容易使用和有很强的扩展性。Netty的内部实现时很复杂的，但是Netty提供了简单易用的api从网络处理代码中解耦业务逻辑。**Netty是完全基于NIO实现的，所以整个Netty都是异步的。**

**ZeroMQ**

ZeroMQ是一种基于**消息队列的多线程网络库**，其对套接字类型、连接处理、帧、甚至路由的底层细节进行抽象，提供跨越多种传输协议的套接字。ZeroMQ是网络通信中新的一层，介于应用层和传输层之间（按照TCP/IP划分），其是一个可伸缩层，可并行运行，分散在分布式系统间。
ZeroMQ定位为：一个简单好用的传输层，像框架一样的一个socket library，他使得Socket编程更加简单、简洁和性能更高。是一个消息处理队列库，可在多个线程、内核和主机盒之间弹性伸缩。ZMQ的明确目标是“成为标准网络协议栈的一部分，之后进入Linux内核”。



## 1.2 Worker内部通信技术（Distruptor）

### 1.2.1 Disruptor是什么

1. 简单理解：Disruptor是一个Queue。Disruptor是实现了“队列”的功能，而且**是一个有界队列**。而队列的应用场景自然就是“生产者-消费者”模型。
2. 在JDK中Queue有很多实现类，包括不限于ArrayBlockingQueue、LinkBlockingQueue，这两个底层的数据结构分别是数组和链表。数组查询快，链表增删快，能够适应大多数应用场景。
3. 但是ArrayBlockingQueue、LinkBlockingQueue都是线程安全的。涉及到线程安全，就会有synchronized、lock等关键字，这就意味着CPU会打架。 
4. Disruptor是**一种线程之间信息无锁的交换方式**（使用CAS（Compare And Swap/Set）操作）。


### 1.2.2 Disruptor主要特点

1. **没有竞争=没有锁**=非常快。
2. 所有访问者都记录自己的序号的实现方式，允许**多个生产者与多个消费者共享相同的数据结构**。
3. 在每个对象中都能跟踪序列号（ring buffer，claim Strategy，生产者和消费者），加上神奇的cache line padding，就意味着没有为伪共享和非预期的竞争。


### 1.2.3 Disruptor核心技术点

1. Disruptor可以看成一个**事件监听或消息机制**，在队列中一边**生产者**放入消息，另外一边**消费者**并行取出处理.
2. 底层是单个数据结构：一个**ring buffer**。
3. 每个生产者和消费者都有一个**次序计算器**，以显示当前**缓冲**工作方式。
4. 每个生产者消费者能够**操作自己的次序计数器**、能够**读取对方的计数器**，生产者能够读取消费者的计算器确保其在没有锁的情况下是可写的。


**核心组件**

1. **Ring Buffer** 环形的**缓冲区**，负责对通过 Disruptor 进行**交换的数据（事件）进行存储和更新**。
2. **Sequence** 通过顺序**递增的序号**来编号管理通过其进行交换的数据（事件），对数据(事件)的处理过程总是沿着序号逐个递增处理。
3. **RingBuffer** 底层是个**数组**，**次序计算器**是一个**64bit long** 整数型，平滑增长。



![](15134932501917.jpg)


1. 接受数据并写入到脚标31的位置，之后会沿着序号一直写入，但是不会绕过消费者所在的脚标。
2. Joumaler和replicator同时读到24的位置，他们可以批量读取数据到30
3. 消费逻辑线程读到了14的位置，但是没法继续读下去，因为他的sequence暂停在15的位置上，需要等到他的sequence给他序号。如果sequence能正常工作，就能读取到30的数据。



# 2 Storm组件本地目录树


![](15134936856884.jpg)



# 3 Storm ZooKeeper目录树

![](15134938201737.jpg)


# 4 Storm任务提交的过程

![](15134939660539.jpg)


![](15134939939920.jpg)


# 5 Storm消息容错机制



1. 在storm中，可靠的信息处理机制是**从spout开始**的。
2. 一个提供了可靠的处理机制的**spout需要记录他发射出去的tuple**，当下游bolt处理tuple或者子tuple失败时spout能够**重新发射**。 
3. Storm通过调用**Spout的nextTuple()发送一个tuple**。为实现可靠的消息处理，首先要给**每个发出的tuple带上唯一的ID**，并且将ID作为参数传递给SpoutOutputCollector的emit()方法：collector.emit(new Values("value1","value2"), msgId); messageid就是用来标示唯一的tuple的，而rootid是随机生成的
4. 给每个tuple指定ID告诉Storm系统，**无论处理成功还是失败，spout都要接收tuple树上所有节点返回的通知**。如果处理成功，spout的ack()方法将会对编号是msgId的消息应答确认；如果处理失败或者超时，会调用fail()方法。 




