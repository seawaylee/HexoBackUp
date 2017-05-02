---
title: Hadoop学习笔记（一）- Hadoop快速入门
date: 2017-04-24 15:08:31
tags: [大数据,Hadoop]
---

# 1 Hadoop生态圈简介

![](Hadoop学习笔记（一）-%20Hadoop快速入门/hadoop_sys.jpg)

**重点组件：**

- HDFS：分布式文件系统
- MAPREDUCE：分布式运算程序开发框架
- HIVE：基于大数据技术（文件系统+运算框架）的SQL数据仓库工具
- HBASE：基于HADOOP的分布式海量数据库
- ZOOKEEPER：分布式协调服务基础组件
- Mahout：基于mapreduce/spark/flink等分布式运算框架的机器学习算法库
- Oozie：工作流调度框架
- Sqoop：数据导入导出工具
- Flume：日志数据采集框架

<!--more-->

# 2 搭建Hadoop集群
## 2.1 下载、解压、配置环境变量
```command
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.6.4/hadoop-2.6.4.tar.gz 

```
[配置免密登陆](https://seawaylee.github.io/2017/03/22/%E5%A4%A7%E6%95%B0%E6%8D%AE/%E5%A4%A7%E6%95%B0%E6%8D%AE%E5%9F%BA%E7%A1%80/Linux%E7%8E%AF%E5%A2%83%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C/)
## 2.2 Hadoop配置

**hadoop-env.sh**


```
export JAVA_HOME=/home/hadoop/apps/jdk1.7.0_51
```

**core-site.xml**

```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9000</value>
</property>

<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/app/data/hadoop_data</value>
</property>
```

**hdfs-site.xml**

```
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>

```

**mapred-site.xml**   
 
```

<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

**yarn-site.xml**

```
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop1</value>
</property>

<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

**slaves**

```
hadoop1
hadoop2
hadoop3
```

# 3 Hadoop Shell 命令

## 3.1 集群相关

- start-dfs.sh 启动hdfs
- start-yarn.sh 启动yarn
- start-all.sh 直接启动hdfs + yarn
- stop-all.sh 停止 hdfs + yarn
- hadoop namenode -format 

## 3.2 HDFS


| 命令  | 功能 | 示例 |  备注 |
| --- | --- | --- | --- |
| -help               | 输出命令参数手册 |  |  | 
| -ls | 显示目录信息 | hadoop fs -ls /| | 
| -mkdir |创建目录  | hadoop fs -mkdir -p /aaa/bb/cc/dd  | | 
| -moveFromLocal |从本地剪切到hdfs  | hadoop fs -moveFromLocal /home/hadoop/1.txt /aaa/bb/cc/dd| | 
| -moveToLocal | 从hdfs剪切到本地 | hadoop fs -moveToLocal /aaa/bb/cc/dd /home/hadoop/1.txt | | 
| -appendToFile | 追加一个文件到已经存在的文件末尾 | hadoop fs -appendToFile ./hello.txt /hello.txt | | 
| -cat | 查看文件内容 |hadoop fs -cat /hello.txt  | | 
| -tail |显示一个文件的末尾  |hadoop fs -tail /access_log.1  | | 
| -text |以字符形式打印一个文件的内容  |hadoop fs -text /access_log.1  | | 
| -chgrp/-chmod/-chown |修改文件权限  | hadoop fs -chmod 755 /hello.txt | | 
| -copyFromLocal | 从本地文件系统中拷贝文件到hdfs路径去 |  | | 
| -copyToLocal |从hdfs拷贝到本地  |  | | 
| -cp | 从hdfs的一个路径拷贝到hdfs另一个路径 |  | | 
| -mv | 在hdfs中移动文件 |  | | 
| -get | 等同于copyToLocal |  | | 
| -put | 等同于copyFromLocal |  | | 
| -getmerge | 合并下载多个文件 |  | | 
| -rm | 删除文件或文件夹 | hadoop fs -rm -r /aaa/bbb | | 
| -rmdir | 删除空目录 | hadoop fs -rmdir /aaa/bbb/ccc | | 
| -df | 统计问价那系统的可用空间信息 |hadoop fs -df -h /  | | 
| -du |统计文件夹的大小信息  | hadoop fs -du -s -h /aaa/* | | 
| -count | 统计一个指定目录下的文件节点数量 | hadoop fs -count /aaa/ | | 
| -setrep | 设置hdfs中文件的副本数量 | hadoop fs -setrep 3 /aaa/jdk.tar | | 



# 4 使用Hadoop集群
## 4.1 查看集群状态
**`hdfs dfsadmin -report`**

```
Configured Capacity: 64389906432 (59.97 GB)
Present Capacity: 38095904768 (35.48 GB)
DFS Remaining: 38095818752 (35.48 GB)
DFS Used: 86016 (84 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0

-------------------------------------------------
Live datanodes (3):

Name: 10.5.151.242:50010 (hadoop2)
Hostname: localhost
Decommission Status : Normal
Configured Capacity: 21463302144 (19.99 GB)
DFS Used: 28672 (28 KB)
Non DFS Used: 8654106624 (8.06 GB)
DFS Remaining: 12809166848 (11.93 GB)
DFS Used%: 0.00%
DFS Remaining%: 59.68%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Tue Apr 25 19:30:28 CST 2017


Name: 10.5.151.241:50010 (hadoop1)
Hostname: localhost
Decommission Status : Normal
Configured Capacity: 21463302144 (19.99 GB)
DFS Used: 28672 (28 KB)
Non DFS Used: 9653018624 (8.99 GB)
DFS Remaining: 11810254848 (11.00 GB)
DFS Used%: 0.00%
DFS Remaining%: 55.03%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Tue Apr 25 19:30:28 CST 2017


Name: 10.5.151.243:50010 (hadoop3)
Hostname: localhost
Decommission Status : Normal
Configured Capacity: 21463302144 (19.99 GB)
DFS Used: 28672 (28 KB)
Non DFS Used: 7986876416 (7.44 GB)
DFS Remaining: 13476397056 (12.55 GB)
DFS Used%: 0.00%
DFS Remaining%: 62.79%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Tue Apr 25 19:30:27 CST 2017

```
或者打开 http://hadoop1:50070查看
![](Hadoop学习笔记（一）-%20Hadoop快速入门/hdfs.jpg)
## 4.2 上传、下载文件

HDFS默认文件大小块为128MB，若文件大于128MB，HDFS将会对文件进行切割后存储。
HDFS对文件执行的切割非常简单，我们可以将下载后的多个切块使用cat block1 >> tmp.txt ,cat block2 >> tmp.txt 命令将完整的文件拼接出来。

HDFS文件的增删改查可以使用Hadoop的Shell命令进行操作。

## 4.3 测试MapReduce

- 上传文本文件到hdfs  /paper.txt
- cd /opt/app/hadoop-2.6.4/share/hadoop/mapreduce
- hadoop jar hadoop-mapreduce-examples-2.6.4.jar wordcount /paper.txt /wordcount/output 
- 到 http://hadoop1:8088 查看job执行流程
- 到HDFS查看统计结果  hadoop fs -cat /wordcount/output/part-r-00000





# 5 问题解决记录

##  5.1 CentOS7 64位系统问题
**启动报错**

```
WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

```

**解决**

vim /etc/profile


```
JAVA_HOME=/usr/java/jdk1.7.0_79
CLASSPATH=.:$JAVA_HOME/lib.tools.jar
ZOOKEEPER_HOME=/opt/app/zookeeper-3.4.5
HADOOP_HOME=/opt/app/hadoop-2.6.4
PATH=$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$ZOOKEEPER_HOME/bin:$JAVA_HOME/bin:$PATH
export HADOOP_COMMON_LIB_NATIVE_DIR="$HADOOP_HOME/lib/native/"
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/"
export HADOOP_HOME  ZOOKEEPER_HOME JAVA_HOME CLASSPATH PATH
```

---



## 5.2 格式化DFS

**报错**

```
UnknownHostException - Formatting HDFS on Mac OSX Mavericks
```

**解决**

vim /etc/host
添加主机名 10-5-151-241

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 10-5-151-241
```


## 5.3 启动DataNode失败

**报错**

```
org.apache.hadoop.hdfs.server.common.Storage: DataNode version: -56 and NameNode layout version: -60
```

**解决**

根据日志中的路径，cd /home/hadoop/tmp/dfs

能看到 data和name两个文件夹，

将name/current下的VERSION中的clusterID复制到data/current下的VERSION中，覆盖掉原来的clusterID
 
让两个保持一致

然后重启，启动后执行jps，查看进程

**或 直接删除data文件夹**

