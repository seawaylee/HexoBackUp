---
title: Storm学习笔记(一) - 基础篇
date: 2017-11-29 01:05:14
tags: [大数据,Storm]
---


## 1 概述


### 1.1 离线计算、流式计算、实时计算 概述

**什么是离线计算？**

- 离线计算：批量获取数据、批量传输数据、**周期性**批量计算数据、展示数据
- 代表技术：Sqoop批量导入数据、HDFS批量存储数据、MapReduce批量计算数据、Hive批量计算数据、Azkaban任务调度

**什么是流式计算？**

- 流式计算：数据**实时产生、实时传输、实时计算、实时展示**
- 代表数据：Flume实时获取数据、Kafka实时数据存储、Storm/JStrom实时数据计算、Redis实时结果缓存、持久化存储（Mysql）
- 总结：将源源不断产生的数据实时收集并实时计算

**流式计算一般架构图**

![](15118787232094.jpg)/


**离线计算与实时计算最大的区别？**

- 收集
- 计算
- 展示


### 1.2 Storm概述

- Storm用来实时处理数据
- 使用场景
    - 日志分析
    - 管道系统
    - 消息转化器
- 特点
    - 低延迟
    - 高可用
    - 分布式
    - 可扩展
    - 数据不丢失
    - 简单易用的接口

**Storm与Hadoop对比**

* Storm用于实时计算，Hadoop用于离线计算。
* Storm处理的数据保存在内存中，源源不断；Hadoop处理的数据保存在文件系统中，一批一批。
* Storm的数据通过网络传输进来；Hadoop的数据保存在磁盘中。
* Storm与Hadoop的编程模型相似

![](15118782566404.jpg)

<!--more-->


## 2 Storm核心组件

### 2.1 组件简介

![](15118783786966.jpg)

- Nimbus：负责**资源分配**和**任务调度**
- Supervisor：负责接受Nimbus分配的任务，启动和停止属于**自己管理的worker进程**。
- Worker：运行具体**处理逻辑的进程**。只有两种类型，**Spout和Bolt**。
- Task：Worker中每一个Spout/Bolt的线程成为一个Task。不同的S/B的Task可能会共享一个物理线程，该线程成为Executor。

### 2.2 编程模型

![](15118785708213.jpg)

- Topology：Storm中运行的一个实时应用程序的名称。
- Spout：在一个Topology中**获取源数据流**的组件。
- Bolt：接受数据然后**执行处理逻辑**的组件。
- Tuple：一次**消息传递的基本单元**，理解为一组消息就是是一个Tuple。
- Stream：表示数据流向。


### 2.3 Storm并发机制

#### 2.3.1 概念

* Workers (JVMs): 在一个物理节点上可以运行**一个或多个独立的JVM进程**。一个Topology可以包含一个或多个worker(并行的跑在不同的物理机上), 所以worker process就是执行一个topology的子集, 并且worker只能对应于一个topology
* Executors (threads): **在一个worker JVM进程中运行着多个Java线程。一个executor线程可以执行一个或多个tasks。**但一般默认每个executor只执行一个task。一个worker可以包含一个或多个executor, 每个component (spout或bolt)至少对应于一个executor, 所以可以说executor执行一个compenent的子集, 同时一个executor只能对应于一个component。 
* Tasks(bolt/spout instances)：Task就是具体的处理逻辑对象，每一个Spout和Bolt会被当作很多task在整个集群里面执行。**每一个task对应到一个线程**，而stream grouping则是定义怎么从一堆task发射tuple到另外一堆task。你可以调用**TopologyBuilder.setSpout和TopBuilder.setBolt来设置并行度 — 也就是有多少个task。 **

#### 2.3.2 配置并行度

* 对于并发度的配置, 在storm里面可以在多个地方进行配置, 优先级为：
    - defaults.yaml < storm.yaml < topology-specific configuration < internal component-specific configuration < external component-specific configuration 
* worker processes的数目, 可以通过配置文件和代码中配置, worker就是执行进程, 所以考虑并发的效果, **数目至少应该大亍machines的数目** 
* executor的数目, **component的并发线程数**，只能在代码中配置(通过setBolt和setSpout的参数), 例如, setBolt("green-bolt", new GreenBolt(), 2) 
* tasks的数目, 可以不配置, **默认和executor1:1**, 也可以通过setNumTasks()配置 
* Topology的worker数通过config设置，即执行该topology的worker（java）进程数。它可以通过 storm rebalance 命令任意调整。 

**代码配置**

```java
Config conf = newConfig();
conf.setNumWorkers(2); //用2个worker
topologyBuilder.setSpout("blue-spout", newBlueSpout(), 2); //设置2个并发度
topologyBuilder.setBolt("green-bolt", newGreenBolt(), 2).setNumTasks(4).shuffleGrouping("blue-spout"); //设置2个并发度，4个任务
topologyBuilder.setBolt("yellow-bolt", newYellowBolt(), 6).shuffleGrouping("green-bolt"); //设置6个并发度
StormSubmitter.submitTopology("mytopology", conf, topologyBuilder.createTopology());
```

**图解**

![](15118904613344.jpg)

**代码配置对应的策略**

- 3个组件的并发度加起来是10，就是说拓扑一共有10个executor
- 一共有2个worker，每个worker产生10 / 2 = 5条线程
- 绿色的bolt配置成2个executor和4个task。为此每个executor为这个bolt运行2个task。

**动态的改变并行度**

- Storm支持在不 restart topology 的情况下, 动态的改变(增减) worker processes 的数目和 executors 的数目, 称为rebalancing. 通过Storm web UI，或者通过storm rebalance命令实现： 
- storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10


## 3 部署和测试

### 3.1 安装、部署

#### 3.1.1 单机伪分布式部署

1. 下载storm-1.1.1压缩包
2. 解压、配置环境变量
3. 修改配置文件

_storm.yaml_


```
#指定storm使用的zk集群
storm.zookeeper.servers:
     - "127.0.0.1"
#指定storm本地状态保存地址
storm.local.dir: "~/app/storm-1.1.1/workdir"
#指定storm集群中的nimbus节点所在的服务器
nimbus.host: "127.0.0.1"
#指定nimbus启动JVM最大可用内存大小
nimbus.childopts: "-Xmx1024m"
#指定supervisor启动JVM最大可用内存大小
supervisor.childopts: "-Xmx1024m"
#指定supervisor节点上，每个worker启动JVM最大可用内存大小
worker.childopts: "-Xmx768m"
#指定ui启动JVM最大可用内存大小，ui服务一般与nimbus同在一个节点上。
ui.childopts: "-Xmx768m"
#指定supervisor节点上，启动worker时对应的端口号，每个端口对应槽，每个槽位对应一个worker
supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703

```



#### 3.1.2 启动集群

1. `zhServer.sh start`  先启动ZooKeeper
2. `nohup storm nimbus &` 在nimbus.host机器上启动nimbus服务
3. `nohup storm ui &` 在nimbus.host机器上启动UI服务
4. `nohup strom supervisor` 在其他节点上启动supervisor服务（单机就直接在host上启动就ok）

#### 3.1.3 查看集群

**访问 http://nimbus.host:8080**

![](15118851356834.jpg)




**查看Storm日志**

- nimbus: *$STORM_HOME/logs/nimbus.log*
- ui: *$STORM_HOME/logs/ui.log*
- supervisor: *$STORM_HOME/logs/supervisor.log*
- worker(在某台supervisor上): *$STORM_HOME/logs/workers-artifacts/wordcount-1-1511791820（Task名称）/6700(Worker所在端口号)/worker.log*

### 3.2 测试集群

#### 3.2.1 Storm集群常用命令

**有许多简单且有用的命令可以用来管理拓扑，它们可以提交、杀死、禁用、再平衡拓扑（Topology）。**

- 提交任务命令格式：storm jar 【jar路径】 【拓扑包名.拓扑类名】 【拓扑名称】
    - bin/storm jar examples/storm-starter/storm-starter-topologies-0.9.6.jar storm.starter.WordCountTopology wordcount
- 杀死任务命令格式：storm kill 【拓扑名称】 -w 10（执行kill命令时可以通过-w [等待秒数]指定拓扑停用以后的等待时间）
    - storm kill topology-name -w 10
- 停用任务命令格式：storm deactivte  【拓扑名称】
    - storm deactivte topology-name
    - 我们能够挂起或停用运行中的拓扑。当停用拓扑时，所有已分发的元组都会得到处理，但是spouts的nextTuple方法不会被调用。销毁一个拓扑，可以使用kill命令。它会以一种安全的方式销毁一个拓扑，首先停用拓扑，在等待拓扑消息的时间段内允许拓扑完成当前的数据流。
- 启用任务命令格式：storm activate【拓扑名称】
    - storm activate topology-name
- 重新部署任务命令格式：storm rebalance  【拓扑名称】
    - storm rebalance topology-name
    - 再平衡使你重分配集群任务。这是个很强大的命令。比如，你向一个运行中的集群增加了节点。再平衡命令将会停用拓扑，然后在相应超时时间之后重分配工人，并重启拓扑。

##### 3.2.2 运行WordCount-Topology

`$ storm jar storm-starter.jar org.apache.storm.starter.WordCountTopology theirWordCount`

![](15118849400727.jpg)

    

**运行后集群状态**

![](15118850264899.jpg)


## 4 第一个Storm程序


### 4.1 WordCount



```java
package storm.word_count;

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

import java.util.HashMap;
import java.util.Map;
import java.util.Random;

/**
 * @author NikoBelic
 * @create 2017/11/29 00:26
 */
public class WordCountTopology
{
    public static class MySpout extends BaseRichSpout
    {

        SpoutOutputCollector collector;
        String[] sourceData = new String[]{
                "Nowadays, going to gym after work is most people’s choice, because they need to be keep fit.",
                "The young blue and white collar sit in their offices all the daytime and they lack of exercise,",
                "most of them are in the sub-health state.",
                "They realize the importance of taking exercise and more gyms have been opened to meet their needs.",
                "In my opinion, we can not only exercise our body, but also to have fun.",
                "The gym provides people a place to make friends.",
                "They can find someone who has the same interest with them, and then just talk to them.",
                "They can forget about the annoyance and just relax themselves.",
                "Some girls like to do yoga while some boys like to play badmiton.",
                "They can choose whatever they like, which is good for keeping good mood."

        };

        /**
         * 初始化方法
         *
         * @Author SeawayLee
         * @Date 2017/11/29 00:41
         */
        @Override
        public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector)
        {
            this.collector = spoutOutputCollector;
        }

        /**
         * Storm框架持续无限调用此方法
         *
         * @Author SeawayLee
         * @Date 2017/11/29 00:42
         */
        @Override
        public void nextTuple()
        {
            collector.emit(new Values(sourceData[new Random().nextInt(sourceData.length)]));
            try
            {
                Thread.sleep(1000);
            } catch (InterruptedException e)
            {
                e.printStackTrace();
            }
        }

        /**
         * 定义emit出去的value的field名称
         *
         * @Author SeawayLee
         * @Date 2017/11/29 00:48
         */
        @Override
        public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer)
        {
            outputFieldsDeclarer.declare(new Fields("sentence"));
        }
    }

    public static class MySplitBolt extends BaseRichBolt
    {

        OutputCollector collector;

        /**
         * 初始化方法
         *
         * @Author SeawayLee
         * @Date 2017/11/29 00:46
         */
        @Override
        public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector)
        {
            this.collector = outputCollector;
        }

        /**
         * 被Storm框架持续无限调用此方法
         *
         * @Author SeawayLee
         * @Date 2017/11/29 00:46
         */
        @Override
        public void execute(Tuple tuple)
        {
            String line = tuple.getString(0);
            String[] words = line.split(" ");
            for (String word : words)
            {
                collector.emit(new Values(word, 1));
            }
        }

        /**
         * 定义emit出去的value的field名称
         *
         * @Author SeawayLee
         * @Date 2017/11/29 00:49
         */
        @Override
        public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer)
        {
            outputFieldsDeclarer.declare(new Fields("word", "num"));
        }
    }

    public static class MyCountBold extends BaseRichBolt
    {
        OutputCollector collector;
        Map<String, Integer> map = new HashMap<>();

        @Override
        public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector)
        {
            this.collector = outputCollector;
        }

        @Override
        public void execute(Tuple tuple)
        {
            String word = tuple.getString(0);
            Integer count = tuple.getInteger(1);
            System.out.println(Thread.currentThread().getId() + "   word:" + word);
            map.put(word, map.containsKey(word) ? map.get(word) + count : count);
            System.out.println("count:" + map);
        }

        @Override
        public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer)
        {
            // 不输出
        }
    }

    public static void main(String[] args)
    {
        // 1. 准备Topology
        TopologyBuilder topologyBuilder = new TopologyBuilder();
        topologyBuilder.setSpout("mySpout", new MySpout(), 2);
        topologyBuilder.setBolt("splitBolt", new MySplitBolt(), 2).shuffleGrouping("mySpout");
        topologyBuilder.setBolt("countBolt", new MyCountBold(), 4).fieldsGrouping("splitBolt", new Fields("word"));

        // 2. 创建Config，指定当前Topology需要的Worker数量
        Config config = new Config();
        config.setNumWorkers(2);

        // 3. 提交任务   本地模式 和 集群模式
        LocalCluster localCluster = new LocalCluster();
        localCluster.submitTopology("myWordCount", config, topologyBuilder.createTopology());
        //StormSubmitter.submitTopology("myWordCount", config, topologyBuilder.createTopology());

    }
}

```

![](15118883711684.jpg)

**注意：我们必须使用FieldsGrouping才能正确的统计出每个单词的总数，将hash(word)值相同的单词分配到同一个CountBolt。**


### 4.2 StreamGrouping详解


**Storm里面有7种类型的stream grouping**

- **Shuffle Grouping**: 随机分组， **随机派发stream里面的tuple**，保证每个bolt接收到的tuple数目大致相同。
- **Fields Grouping**：按字段分组，比如按userid来分组，**具有同样userid的tuple会被分到相同的Bolts里的一个task**，而不同的userid则会被分配到不同的bolts里的task。
* **All Grouping**：**广播发送**，对于每一个tuple，所有的bolts都会收到。
* **Global Grouping**：全局分组， 这个tuple被分配到storm中的一个bolt的其中一个task。再具体一点就是分配给id值最低的那个task。
* **Non Grouping**：不分组，这stream grouping个分组的意思是说stream不关心到底谁会收到它的tuple。目前这种分组**和Shuffle grouping是一样的效果**， 有一点不同的是storm会把这个bolt放到这个bolt的订阅者同一个线程里面去执行。
* **Direct Grouping**： 直接分组， 这是一种比较特别的分组方法，用这种分组意味着消息的发送者指定由消息接收者的哪个task处理这个消息。只有被声明为Direct Stream的消息流可以声明这种分组方法。而且这种消息tuple必须使用emitDirect方法来发射。消息处理者可以通过TopologyContext来获取处理它的消息的task的id （OutputCollector.emit方法也会返回task的id）。
* **Local or shuffle grouping**：如果目标bolt有一个或者多个task在**同一个工作进程中**，tuple将会被随机发生给这些tasks。否则，和普通的Shuffle Grouping行为一致。



