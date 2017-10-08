---
title: Hadoop学习笔记（六）- Hadoop的HA原理与配置
date: 2017-06-13 20:03:06
tags: [大数据,Hadoop]
---


# 1 Hadoop的HA机制

正式引入HA机制是从hadoop2.0开始，之前的版本中没有HA机制。

Hadoo的HA机制可以分成各个组件的HA机制

- HDFS的HA
- YARN的HA

## 1.1 HDFS的HA

- 通过双NameNode消除单点故障
- 双NameNode工作要点如下：
    - A. **元数据管理**方式需要改变
        1. 双NameNode各自在内存中保存一份元数据(fsimage+edits合成结果)
        2. edits日志只能有一份，只有Active状态的NameNode节点可以对其进行写操作
        3. 双NameNode都可以读取edits
        4. 共享的edits仿造一个共享的存储介质中管理（qjournal和NFS两个主流实现）
    - B. 需要一个**状态管理**功能模块
        1. 实现了一个ZKFC（ZooKeeperFailContrl），常驻在**每一个NameNode**所在的节点
        2. 每一个ZKFC负责监控自己所在的NameNode节点，利用ZK进行**状态标识**
        3. 当需要进行状态切换时，由**ZKFC来负责切换**（RPC调用NameNode中的方法）
        4. 切换时需要放置**BrainSplit**（多个NameNode都是Active状态）现象的发生（新Active节点主动发送杀死被取代Active状态的节点的命令，**防止假死**）

    ![](HDFS_HA.png)
    
    
<!--more-->

## 1.2 Yarn的HA

- 双ResourceManager消除单点故障
- 双RM工作要点如下：
    - A. 直接由ZooKeeper检测双节点状态。
    - B. 假设Application在请求资源的过程中，当前Active状态的RM发生故障，则直接重新发送请求，并不产生严重影响。

    ![](Yarn_HA.png)
    
# 2 HA环境搭建

## 2.1 集群节点规划

### 2.1.1 10节点规划

| 节点名称 | 节点服务 | 备注  |
| --- | --- | --- |
| Server01 | NameNode、ZKFC | start-dfs.sh |
| Server02 | NameNode、ZKFC |  |
| Server03 | ResourceManager |  |
| Server04 | ResourceManager |  |
| Server05 | DataNode、NodeManager | start-yarn.sh |
| Server06 | DataNode、NodeManager |  |
| Server07 | DataNode、NodeManager |  |
| Server08 | JournalNode、ZooKeeper |  |
| Server09 | JournalNode、ZooKeeper |  |
| Server10 | JournalNode、ZooKeeper |  |

### 2.1.2 3节点规划

| 节点名称 | 节点服务 | 备注  |
| --- | --- | --- |
| Server01 | NameNode、ZKFC、ResourceManager、NodeManager、DataNode、ZooKeeper、JournalNode | |
| Server02 | NameNode、ZKFC、ResourceManager、NodeManager、DataNode、ZooKeeper、JournalNode |  |
| Server03 | DataNode、NodeManager、ZooKeeper、JournalNode |  |


## 2.2 配置文件

**core-site.xml**

```xml
<configuration>
	<!-- 指定hdfs的nameservice为ns1 -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://ns1/</value>
	</property>
	<!-- 指定hadoop临时目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/home/hadoop/app/hadoop-2.4.1/tmp</value>
	</property>
	
	<!-- 指定zookeeper地址 -->
	<property>
		<name>ha.zookeeper.quorum</name>
		<value>weekend05:2181,weekend06:2181,weekend07:2181</value>
	</property>
</configuration>
```


**hdfs-site.xml**


```xml
configuration>
<!--指定hdfs的nameservice为ns1，需要和core-site.xml中的保持一致 -->
<property>
	<name>dfs.nameservices</name>
	<value>ns1</value>
</property>
<!-- ns1下面有两个NameNode，分别是nn1，nn2 -->
<property>
	<name>dfs.ha.namenodes.ns1</name>
	<value>nn1,nn2</value>
</property>
<!-- nn1的RPC通信地址 -->
<property>
	<name>dfs.namenode.rpc-address.ns1.nn1</name>
	<value>weekend01:9000</value>
</property>
<!-- nn1的http通信地址 -->
<property>
	<name>dfs.namenode.http-address.ns1.nn1</name>
	<value>weekend01:50070</value>
</property>
<!-- nn2的RPC通信地址 -->
<property>
	<name>dfs.namenode.rpc-address.ns1.nn2</name>
	<value>weekend02:9000</value>
</property>
<!-- nn2的http通信地址 -->
<property>
	<name>dfs.namenode.http-address.ns1.nn2</name>
	<value>weekend02:50070</value>
</property>
<!-- 指定NameNode的edits元数据在JournalNode上的存放位置 -->
<property>
	<name>dfs.namenode.shared.edits.dir</name>
	<value>qjournal://weekend05:8485;weekend06:8485;weekend07:8485/ns1</value>
</property>
<!-- 指定JournalNode在本地磁盘存放数据的位置 -->
<property>
	<name>dfs.journalnode.edits.dir</name>
	<value>/home/hadoop/app/hadoop-2.4.1/journaldata</value>
</property>
<!-- 开启NameNode失败自动切换 -->
<property>
	<name>dfs.ha.automatic-failover.enabled</name>
	<value>true</value>
</property>
<!-- 配置失败自动切换实现方式 -->
<property>
	<name>dfs.client.failover.proxy.provider.ns1</name>
	<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
<!-- 配置隔离机制方法，多个机制用换行分割，即每个机制暂用一行-->
<property>
	<name>dfs.ha.fencing.methods</name>
	<value>
		sshfence
		shell(/bin/true)
	</value>
</property>
<!-- 使用sshfence隔离机制时需要ssh免登陆 -->
<property>
	<name>dfs.ha.fencing.ssh.private-key-files</name>
	<value>/home/hadoop/.ssh/id_rsa</value>
</property>
<!-- 配置sshfence隔离机制超时时间 -->
<property>
	<name>dfs.ha.fencing.ssh.connect-timeout</name>
	<value>30000</value>
</property>
/configuration>
```

# 3 集群运维测试


## 3.1 DataNode动态上下线

Datanode动态上下线很简单，步骤如下：

1. 准备一台服务器，设置好环境
2. 部署hadoop的安装包，并同步集群配置
3. 联网上线，新datanode会自动加入集群
4. 如果是一次增加大批datanode，还应该做集群负载重均衡

## 3.2 NameNode状态切换管理

使用的命令上hdfs  haadmin
可用 hdfs  haadmin –help查看所有帮助信息

可以看到，状态操作的命令示例：

- 查看namenode工作状态   `hdfs haadmin -getServiceState nn1`
- 将standby状态NameNode切换到active `hdfs haadmin –transitionToActive nn1`
- 将active状态NameNode切换到standby `hdfs haadmin –transitionToStandby nn2`

## 3.3 数据块的Balance

- 启动balancer的命令： `start-balancer.sh -threshold 8`
- 运行之后，会有Balancer进程出现。
- 上述命令设置了Threshold为8%，那么执行balancer命令的时候，首先统计所有DataNode的磁盘利用率的均值，然后判断如果某一个DataNode的磁盘利用率超过这个均值Threshold，那么将会把这个DataNode的block转移到磁盘利用率低的DataNode，这对于新节点的加入来说十分有用。Threshold的值为1到100之间，不显示的进行参数设置的话，默认是10。

## 3.4 HA下HDFS-API的变化

客户端需要nameservice的配置信息，其他不变


```java
/**
 * 如果访问的是一个ha机制的集群
 * 则一定要把core-site.xml和hdfs-site.xml配置文件放在客户端程序的classpath下
 * 以让客户端能够理解hdfs://ns1/中  “ns1”是一个ha机制中的namenode对——nameservice
 * 以及知道ns1下具体的namenode通信地址
 * @author
 *
 */
public class UploadFile {
	
	public static void main(String[] args) throws Exception  {
		
		Configuration conf = new Configuration();
		conf.set("fs.defaultFS", "hdfs://ns1/");
		
		FileSystem fs = FileSystem.get(new URI("hdfs://ns1/"),conf,"hadoop");
		
		fs.copyFromLocalFile(new Path("g:/eclipse-jee-luna-SR1-linux-gtk.tar.gz"), new Path("hdfs://ns1/"));
		
		fs.close();
		
	}
	

}
```

