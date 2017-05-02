---
title: Hadoop学习笔记（二） - HDFS详解
date: 2017-05-02 19:14:03
tags: [大数据,Hadoop]
---

# HDFS详解

# 1 HDFS 基本概念

## 1.1 前言
- 设计思想

分而治之：将大文件、大批量文件，分布式存放在大量服务器上，以便于采取分而治之的方式对海量数据进行运算分析；

- 在大数据系统中作用

为各类分布式运算框架（如：mapreduce，spark，tez，……）提供数据存储服务

- 重点概念

文件切块，副本存放，元数据

## 1.2 HDFS概念和特性

首先，它是一个文件系统，用于存储文件，通过统一的命名空间——目录树来定位文件；
其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色；

**重要特性如下：**

- HDFS中的文件在物理上是分块存储（block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，老版本中是64M
- HDFS文件系统会给客户端提供一个统一的抽象目录树，客户端通过路径来访问文件，形如：hdfs://namenode:port/dir-a/dir-b/dir-c/file.data
- 目录结构及文件分块信息(元数据)的管理由namenode节点承担。namenode是HDFS集群主节点，负责维护整个hdfs文件系统的目录树，以及每一个路径（文件）所对应的block块信息（block的id，及所在的datanode服务器）
- 文件的各个block的存储管理由datanode节点承担。datanode是HDFS集群从节点，每一个block都可以在多个datanode上存储多个副本（副本数量也可以通过参数设置dfs.replication）
- HDFS是设计成适应一次写入，多次读出的场景，且不支持文件的修改(注：适合用来做数据分析，并不适合用来做网盘应用，因为，不便修改，延迟大，网络开销大，成本太高)

<!--more-->

## 1.3 基本操作

| 命令  | 功能 | 示例 | 
| --- | --- | --- | --- |
| -help | 输出命令参数手册 |  |
| -ls | 显示目录信息 | hadoop fs -ls /| 
| -mkdir |创建目录  | hadoop fs -mkdir -p /aaa/bb/cc/dd  | 
| -moveFromLocal |从本地剪切到hdfs  | hadoop fs -moveFromLocal /home/hadoop/1.txt /aaa/bb/cc/dd| 
| -moveToLocal | 从hdfs剪切到本地 | hadoop fs -moveToLocal /aaa/bb/cc/dd /home/hadoop/1.txt | 
| -appendToFile | 追加一个文件到已经存在的文件末尾 | hadoop fs -appendToFile ./hello.txt /hello.txt | 
| -cat | 查看文件内容 |hadoop fs -cat /hello.txt  | 
| -tail |显示一个文件的末尾  |hadoop fs -tail /access_log.1  | 
| -text |以字符形式打印一个文件的内容  |hadoop fs -text /access_log.1  | 
| -chgrp/-chmod/-chown |修改文件权限  | hadoop fs -chmod 755 /hello.txt | 
| -copyFromLocal | 从本地文件系统中拷贝文件到hdfs路径去 |  | 
| -copyToLocal |从hdfs拷贝到本地  |  | 
| -cp | 从hdfs的一个路径拷贝到hdfs另一个路径 |  | 
| -mv | 在hdfs中移动文件 |  | 
| -get | 等同于copyToLocal |  | 
| -put | 等同于copyFromLocal |  | 
| -getmerge | 合并下载多个文件 |  | 
| -rm | 删除文件或文件夹 | hadoop fs -rm -r /aaa/bbb | 
| -rmdir | 删除空目录 | hadoop fs -rmdir /aaa/bbb/ccc | 
| -df | 统计问价那系统的可用空间信息 |hadoop fs -df -h /  | 
| -du |统计文件夹的大小信息  | hadoop fs -du -s -h /aaa/* | 
| -count | 统计一个指定目录下的文件节点数量 | hadoop fs -count /aaa/ | 
| -setrep | 设置hdfs中文件的副本数量 | hadoop fs -setrep 3 /aaa/jdk.tar | 


# 2 HDFS 原理

## 2.1 概述

1. HDFS集群分为两大角色：NameNode、DataNode
2. NameNode负责管理整个文件系统的元数据
3. DataNode 负责管理用户的文件数据块
4. 文件会按照固定的大小（blocksize）切成若干块后分布式存储在若干台datanode上
5. 每一个文件块可以有多个副本，并存放在不同的datanode上
6. Datanode会定期向Namenode汇报自身所保存的文件block信息，而namenode则会负责保持文件的副本数量
7. HDFS的内部工作机制对客户端保持透明，客户端请求访问HDFS都是通过向namenode申请来进行

## 2.2 HDFS 读写数据详细步骤

### 2.2.1 写数据
![](Hadoop学习笔记（二）-%20HDFS详解/4.jpg)


1. 根namenode通信请求上传文件，namenode检查目标文件是否已存在，父目录是否存在
2. namenode返回是否可以上传
3. client请求第一个 block该传输到哪些datanode服务器上
4. namenode返回3个datanode服务器ABC
5. client请求3台dn中的一台A上传数据（本质上是一个RPC调用，建立pipeline），A收到请求会继续调用B，然后B调用C，将真个pipeline建立完成，逐级返回客户端
6. client开始往A上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位，A收到一个packet就会传给B，B传给C；A每传一个packet会放入一个应答队列等待应答
7. 当一个block传输完成之后，client再次请求namenode上传第二个block的服务器。

### 2.2.2 读数据
![](Hadoop学习笔记（二）-%20HDFS详解/1.jpg)

## 2.3 NameNode工作机制

**NAMENODE职责：**
1、负责客户端请求的响应
2、元数据的管理（查询，修改）

**namenode对数据的管理采用了三种存储形式：**
1、内存元数据(NameSystem)
2、磁盘元数据镜像文件
3、数据操作日志文件（可通过日志运算出元数据）


### 2.3.1 元数据的存储机制

A、内存中有一份完整的元数据(内存meta data)
B、磁盘有一个“准完整”的元数据镜像（fsimage）文件(在namenode的工作目录中)
C、用于衔接内存metadata和持久化元数据镜像fsimage之间的操作日志（edits文件）注：当客户端对hdfs中的文件进行新增或者修改操作，操作记录首先被记入edits日志文件中，当客户端操作成功后，相应的元数据会更新到内存meta.data中

### 2.3.2 元数据的checkpoint
每隔一段时间，会由secondaryNamenode 将 namenode 上积累的所有edits和一个最新的fsimage下载到本地，并架子啊到内存进行merge（这个过程称为checkpoint）

**checkpoint的详细过程**
![](Hadoop学习笔记（二）-%20HDFS详解/2.jpg)


**checkpoint操作的出发条件配置参数**

![](Hadoop学习笔记（二）-%20HDFS详解/3.jpg)

**chekcpoint的附带作用**

namenode和secondary namenode的工作目录存储结构完全相同，所以，当namenode故障退出需要重新恢复时，可以从secondary namenode的工作目录中将fsimage拷贝到namenode的工作目录，以恢复namenode的元数据


### 2.3.3 元数据目录说明

1、在第一次部署好Hadoop集群的时候，我们需要在NameNode（NN）节点上格式化磁盘：
`$HADOOP_HOME/bin/hdfs namenode -format`

2、格式化完成之后，将会在$dfs.namenode.name.dir/current目录下如下的文件结构

```command
current/
|-- VERSION
|-- edits_*
|-- fsimage_0000000000008547077
|-- fsimage_0000000000008547077.md5
`-- seen_txid
```

3、其中的dfs.name.dir是在hdfs-site.xml文件中配置的，默认值如下：


```xml
<property>
  <name>dfs.name.dir</name>
  <value>file://${hadoop.tmp.dir}/dfs/name</value>
</property>

<!--hadoop.tmp.dir是在core-site.xml中配置的，默认值如下-->
<property>
  <name>hadoop.tmp.dir</name>
  <value>/tmp/hadoop-${user.name}</value>
  <description>A base for other temporary directories.</description>
</property>
```

4、fs.namenode.name.dir属性可以配置多个目录，
如/data1/dfs/name,/data2/dfs/name,/data3/dfs/name,....。各个目录存储的文件结构和内容都完全一样，**相当于备份**，这样做的好处是当其中一个目录损坏了，也不会影响到Hadoop的元数据，特别是当其中一个目录是NFS（网络文件系统Network File System，NFS）之上，即使你这台机器损坏了，元数据也得到保存。 
5、下面对$dfs.namenode.name.dir/current/目录下的文件进行解释。

**VERSION文件是Java属性文件，内容大致如下：**

```
namespaceID=934548976
clusterID=CID-cdff7d73-93cd-4783-9399-0a22e6dce196
cTime=0
storageType=NAME_NODE
blockpoolID=BP-893790215-192.168.24.72-1383809616115
layoutVersion=-47
```

- 1、namespaceID是文件系统的唯一标识符，在文件系统首次格式化之后生成的
- 2、storageType说明这个文件存储的是什么进程的数据结构信息（如果是DataNode，storageType=DATA_NODE）
- 3、cTime表示NameNode存储时间的创建时间，由于我的NameNode没有更新过，所以这里的记录值为0，以后对NameNode升级之后，cTime将会记录更新时间戳
- 4、layoutVersion表示HDFS永久性数据结构的版本信息， 只要数据结构变更，版本号也要递减，此时的HDFS也需要升级，否则磁盘仍旧是使用旧版本的数据结构，这会导致新版本的NameNode无法使用
- 5、clusterID是系统生成或手动指定的集群ID，在-clusterid选项中可以使用它；如下说明

    - a、使用如下命令格式化一个Namenode：
    $HADOOP_HOME/bin/hdfs namenode -format [-clusterId <cluster_id>]
    选择一个唯一的cluster_id，并且这个cluster_id不能与环境中其他集群有冲突。如果没有提供cluster_id，则会自动生成一个唯一的ClusterID。
    - b、使用如下命令格式化其他Namenode：
     $HADOOP_HOME/bin/hdfs namenode -format -clusterId <cluster_id>
    - c、升级集群至最新版本。在升级过程中需要提供一个ClusterID，例如：
    $HADOOP_PREFIX_HOME/bin/hdfs start namenode --config $HADOOP_CONF_DIR  -upgrade -clusterId <cluster_ID>
    如果没有提供ClusterID，则会自动生成一个ClusterID。

- 6、blockpoolID：是针对每一个Namespace所对应的blockpool的ID，上面的这个BP-893790215-192.168.24.72-1383809616115就是在我的ns1的namespace下的存储块池的ID，这个ID包括了其对应的NameNode节点的ip地址。 　　

**seen_txid**
 $dfs.namenode.name.dir/current/seen_txid非常重要，是存放transactionId的文件，format之后是0，它代表的是namenode里面的edits_*文件的尾数，namenode重启的时候，会按照seen_txid的数字，循序从头跑edits_0000001~到seen_txid的数字。所以当你的hdfs发生异常重启的时候，一定要比对seen_txid内的数字是不是你edits最后的尾数，不然会发生建置namenode时metaData的资料有缺少，导致误删Datanode上多余Block的资讯。
文件中记录的是edits滚动的序号，每次重启namenode时，namenode就知道要将哪些edits进行加载edits


**current目录**

$dfs.namenode.name.dir/current目录下在format的同时也会生成fsimage和edits文件，及其对应的md5校验文件。


# 3 HDFS API

## 3.1 普通方式

```java

package hadoop.hdfs;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.Iterator;
import java.util.Map;

/**
 * HDFS 基本API使用
 * @author NikoBelic
 * @create 2017/4/25 15:11
 */
public class HDFSTest
{
    FileSystem fs = null;
    Configuration conf = null;
    @Before
    public void init() throws IOException, URISyntaxException, InterruptedException
    {
        /*
         客户端去操作HDFS时，是有一个用户身份的
         默认情况下，HDFS客户端API会从JVM中获取一个参数来作为自己的用户身份，HADOOP_USER_NAME
         也可以在构建客户端FS对象时指定身份
          */
        conf = new Configuration();
        // 拿到一个文件系统操作的客户端实例对象
        fs = FileSystem.get(new URI("hdfs://10.5.151.241:9000"),conf,"root");
    }

    /**
     * 配置文件参数
     * @Author SeawayLee
     * @Date 2017/05/02 12:22
     */
    @Test
    public void testConfig()
    {
        Iterator<Map.Entry<String, String>> iterator = conf.iterator();
        while (iterator.hasNext())
        {
            Map.Entry<String, String> entry = iterator.next();
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
    }

    /**
     * 上传文件
     * @Author SeawayLee
     * @Date 2017/05/02 12:23
     */
    @Test
    public void testUpload() throws IOException, InterruptedException
    {
        fs.copyFromLocalFile(new Path("/Users/lixiwei-mac/百度云同步盘/MAC云存储/电子书/机器学习/机器学习-Mitchell-中文-清晰版.pdf"),new Path("/机器学习-Mitchell-中文-清晰版.pdf"));
        fs.close();
    }

    /**
     * 下载文件
     * @Author SeawayLee
     * @Date 2017/05/02 12:23
     */
    @Test
    public void testDownload() throws Exception
    {
        fs.copyToLocalFile(new Path("/paper.txt"),new Path("/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/data"));
        fs.close();
    }

    /**
     * 创建文件夹
     * @Author SeawayLee
     * @Date 2017/05/02 14:46
     */
    @Test
    public void testMkdir() throws IOException
    {
        boolean mkdirs = fs.mkdirs(new Path("/tesMkdir/aaa/bbb"));
        System.out.println(mkdirs);
    }

    /**
     * 删除文件或文件夹
     * @Author SeawayLee
     * @Date 2017/05/02 14:46
     */
    @Test
    public void testDelete() throws IOException
    {
        boolean delete = fs.delete(new Path("/tesMkdir"), true);
        System.out.println(delete);
    }

    /**
     * 查看文件信息 迭代方式
     * @Author SeawayLee
     * @Date 2017/05/02 14:46
     */
    @Test
    public void testLs() throws IOException
    {
        RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
        while (listFiles.hasNext())
        {
            LocatedFileStatus file = listFiles.next();
            System.out.println("BlockSize:" + file.getBlockSize() / 1024.0 / 1024.0 + "MB");
            System.out.println("Owner:" + file.getOwner());
            System.out.println("Replication:" + file.getReplication());
            System.out.println("Permission:" + file.getPermission());
            System.out.println("Name:" + file.getPath().getName());
            BlockLocation[] blockLocations = file.getBlockLocations();
            System.out.println("");
            for (BlockLocation blockLocation : blockLocations)
            {
                System.out.println("Block-Name:");
                for (String s : blockLocation.getNames())
                {
                    System.out.print(s + " ");
                }
                System.out.println("");
                System.out.println("Block-Offset:" + blockLocation.getOffset());
                System.out.println("Block-Length:" + blockLocation.getLength());
                System.out.println("Block-Hosts:");
                for (String host : blockLocation.getHosts())
                {
                    System.out.print(host + "  ");
                }
                System.out.println("");
            }
            System.out.println("==========================");
        }

    }

    /**
     * 查看文件信息 数组方式
     * @Author SeawayLee
     * @Date 2017/05/02 14:46
     */
    @Test
    public void testLs2() throws IOException
    {
        FileStatus[] fileStatuses = fs.listStatus(new Path("/"));
        for (FileStatus fileStatus : fileStatuses)
        {
            System.out.println(fileStatus.getPath().getName());
        }
    }


}

```

## 3.2 流方式


```java

package hadoop.hdfs;


import org.apache.commons.io.IOUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.junit.Before;
import org.junit.Test;
import sun.nio.ch.IOUtil;

import java.io.*;
import java.net.URI;
import java.net.URISyntaxException;


/**
 * 流方式操作HDFS
 * @author NikoBelic
 * @create 2017/5/2 13:31
 */
public class HdfsStreamAccess
{
    FileSystem fs = null;
    Configuration conf = null;

    @Before
    public void init() throws IOException, URISyntaxException, InterruptedException
    {
        conf = new Configuration();
        fs = FileSystem.get(new URI("hdfs://10.5.151.241:9000"), conf, "root");
    }

    /**
     * 通过流的方式上传文件
     * @Author SeawayLee
     * @Date 2017/05/02 13:41
     */
    @Test
    public void testUpload() throws IOException
    {
        FSDataOutputStream outputStream = fs.create(new Path("/linux系统安装过程.avi"));
        FileInputStream inputStream = new FileInputStream("/Users/lixiwei-mac/linux系统安装过程.avi");
        IOUtils.copy(inputStream, outputStream);

    }

    /**
     * 通过流的方式下载文件
     * @Author SeawayLee
     * @Date 2017/05/02 13:55
     */
    @Test
    public void testDownload() throws IOException
    {
        FSDataInputStream inputStream = fs.open(new Path("/TreeFor.png"));
        FileOutputStream outputStream = new FileOutputStream("/Users/lixiwei-mac/Downloads/fuck.txt");
        IOUtils.copy(inputStream, outputStream);
    }

    /**
     * 通过流的方式指读取文件大小(MapReduce从HDFS读文件进行切片时肯定会用到)
     * @Author SeawayLee
     * @Date 2017/05/02 14:18
     */
    @Test
    public void testRandomAccess() throws IOException
    {
        FSDataInputStream inputStream = fs.open(new Path("/TreeFor.png"));
        inputStream.seek(50);
        InputStreamReader isr = new InputStreamReader(inputStream);
        BufferedReader bf = new BufferedReader(isr);
        char[] buffer = new char[1024 * 20];
        int len = 0;
        while ((len = bf.read(buffer) )!= -1)
        {
            System.out.print(String.valueOf(buffer,0,len));
        }
    }

    /**
     * 流的方式读取文件
     * @Author SeawayLee
     * @Date 2017/05/02 14:21
     */
    @Test
    public void testCat() throws IOException
    {
        FSDataInputStream inputStream = fs.open(new Path("/TreeFor.png"));
        IOUtils.copy(inputStream, System.out);
    }
}



```

# 4 Hadoop中的RPC框架

Hadoop中各个节点的远程调用非常频繁，他自己封装了一套RPC框架，我们也可以直接拿来用，只需要导入Hadoop的common包即可，与Hadoop集群启动与否毫无关系。

> 假设我们需要远程调用一个登陆服务，使用Hadoop的RPC框架很容易就可以实现。

**服务接口**


```java
public interface LoginServiceInterface
{
    public static final long versionID = 1L;

    public String login(String username, String passowrd);
}

```

**服务实现**

```java
public class LoginServiceImpl implements LoginServiceInterface
{
    @Override
    public String login(String username, String passowrd)
    {
        System.out.println(username + ", 你好啊!");

        return username + " Successfully login....";
    }
}

```

**服务启动**


```java
public class PublishServer
{
    public static void main(String[] args) throws IOException
    {
        RPC.Builder builder = new RPC.Builder(new Configuration());
        builder.setBindAddress("localhost")
                .setPort(8888)
                .setProtocol(LoginServiceInterface.class)
                .setInstance(new LoginServiceImpl());
        RPC.Server loginServer = builder.build();
        loginServer.start();
    }
}

```

**客户端远程调用**


```java

public class LoginAction
{
    public static void main(String[] args) throws IOException
    {
        LoginServiceInterface loginService = RPC.getProxy(LoginServiceInterface.class, 2L, new InetSocketAddress("localhost", 8888), new Configuration());
        String res = loginService.login("NikoBelic", "asdasd");
        System.out.println(res);
    }
}

```

**客户端输出**

`NikoBelic, 你好啊!`


