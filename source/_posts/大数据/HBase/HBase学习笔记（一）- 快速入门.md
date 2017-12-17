---
title: HBase学习笔记（一）- 快速入门
date: 2017-11-21 17:25:39
tags: [大数据, HBase]
---

# 1. HBase概述

1. 高可靠性、高性能、面向列、可伸缩的分布式存储系统。
2. 利用HDFS作为其文件存储系统。
3. 利用MapReduce处理数据。
4. 利用ZooKeeper作为协同服务。
5. 与传统数据库对比的优势
    - 线性扩展,数据增多,可以直接增加节点来扩容
    - 数据存储在HDFS,备份机制健全
    - 通过ZK协调查找数据,访问速度快
6. HBase集群中的角色
    - HMaster
    - HRegionServer

**Q1: 为什么会有HBase？**

假设有100W个access.log，每个log大小为1KB，如果使用API向HDFS集群中写，NameNode的压力会很大。
使用HBase可以解决这个问题（文件合并与拆分）

**Q2: HBase存储和HDFS存储的关系?**

- HDFS:  Client -> NameNode -> DataNode

- HBase: Client -> HMaster -> HRegionServer -> Zookeeper(元数据) -> HDFS(数据文件)
    - 每条数据线缓存到HRegionServer的内存中,当达到Block大小时再写入HDFS。
    - HBase中只有表结构和列族,其他都是数据。
    - 1个列族中有多个列,被存储到1个文件中。
    - Hbase中不能删除数据,如果对一个相同的行键插入数据（不同时间戳）,就认为是修改。





<!--more-->
    
# 2. 安装与配置

1. 下载 hbase-1.2.6-bin.tar.gz
2. 解压、配置环境变量
3. 修改配置文件

**hbase-env.sh**
        
```sh
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_71.jdk/Contents/Home
export HBASE_MANAGES_ZK=false
```

**hbase-site.xml**
```xml
<configuration>
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://127.0.0.1:9000/hbase</value>
</property>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>127.0.0.1</value>
</property>
</configuration>
```

**regionservers**
```    
localhost
```

4. 启动

`start-hbase.sh`

5. 监控
    - 进入命令行 `hbase shell`
    - 页面监控 http://master:16010
    
    
    
# 3 HBase 数据模型


![](15111563006180.jpg)


## 3.1 Row Key

1. 与nosql数据库一样,row key 是用来检索记录的主键。
2. 方法Hbase中table的方式
    - 通过 row key 访问
    - 通过 row key 的 range(正则)
    - 全表扫描
3. RowKey 可以使任意字符串（最大长度64KB）
4. 在HBase内部,rk保存为字节数组。
5. 存储时,数据按照rk的字典序排序存储。
6. 设计key时,要充分考虑 排序存储这个特性,将经常一起读取的行存储在一起。


## 3.2 Column Family

- Hbase表中的每个列,都归属于某个列族。
- 列族是schema的一部分（而列不是）,必须在使用表之前定义。
- 列表都以列族为前缀,例如 course:histroy, course:math 都属于course这个列族。

## 3.3 Cell

- 由 rowKey + columnFamily + version 唯一确定的单元。
- 字节码形式存储

## 3.4 Timestamp

- 每个cell都保存着同一份数据的多个版本
- 版本通过时间戳来索引。时间戳的类型是64位整型。
- 时间戳可以由HBASE(在数据写入时自动)赋值,此时时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显式赋值。如果应用程序要避免数据版本冲突,就必须自己生成具有唯一性的时间戳。每个cell中,不同版本的数据按照时间倒序排序,即最新的数据排在最前面。
- 为了避免数据存在过多版本造成的的管理(包括存贮和索引)负担,HBASE提供了两种数据版本回收方式。一是保存数据的最后n个版本,二是保存最近一段时间内的版本（比如最近七天）。用户可以针对每个列族进行设置

# 4 Hbase命令

hbase shell 进入Hbase命令管理界面


| 名称 | 命令行表达式 |
| --- | --- |
| 创建表 | create '表名','列族1','列族2','列族N' | 
| 查看所有表 | list | 
| 描述表 | describe '表名' | 
| 判断表存在 | exists '表名' | 
| 判断是否禁用启用表 | is_enabled/is_disable '表名' | 
| 添加记录 | put '表名','rowKey','列族:列','值' | 
| 查看记录rowkey下的所有数据 | get '表名','rowkey' | 
| 查看表中记录总数 | count 表名 | 
| 获取某个列族 | get '表名','rowkey','列族' | 
| 获取某个列族的某个列 | get '表名','rowkey','列族','列' | 
| 删除记录 | delete '表名','行名','列族:列' | 
| 删除整行 | deleteall '表名','rowkey' | 
| 删除一张表 | 1. disable '表名' 2. drop '表名'  | 
| 清空表 | truncate '表名' | 
| 查看所有记录 | scan '表名' | 
| 查看某个表某个列中的所有数据 |scan '表名',{COLUMNS=> '列族名:列名'} | 
| 更新记录 | 就是重写一遍，进行覆盖，hbase没有修改，都是追加 | 




# 5 HBase依赖的ZooKeeper

## 5.1 ZK在HBase中的功能

- 保存HMaster的地址和 backup-master 地址
- HMaster的功能：
    - 管理HRegionServer
    - 做CRUD的节点
    - 管理HRegionServer中的表分配
- 保存 表-ROOT- 的地址 （Hbase默认的根表，检索表）
- HRegionServer列表
    - 表的CRUD数据
    - 和HDFS交互，读写数据



# 6 HBase原理

## 6.1 体系图

![](15111709041938.jpg)


## 6.2 写流程

1. Client向HRegionServer发送写请求
2. HRegionServer将数据写到Hlog(write ahead log)。为了数据的持久化和恢复
3. HRegionServer将数据写到内存(memstore)
4. 反馈Client写成功

## 6.3 数据Flush过程

1. 当memstore数据达到阀值（默认64M），将数据刷到硬盘，将内存中的数据删除，同事删除HLog中的历史数据。
2. 将数据存储到HDFS中
3. 在Hlog中做标记点

## 6.4 数据合并过程

1. 当数据块达到4块，HMaster将数据加载到本地，进行合并
2. 当合并的数据超过256M，进行拆分，将拆分后的region分配给不同的HRegionServer管理
3. 当HRegionServer宕机后，将HRegionServer上的Hlog拆分，然后分配给不同的HRegionServer加载，修改 .META.
4. 注意：Hlog会同步到HDFS

## 6.5 HBase的读流程

1. 通过ZK和 -ROOT- .META. 表定位HRegionServer
2. 数据从内存和硬盘合并后返回给Client
3. 数据块会缓存


## 6.6 HMaster的职责

1. 管理用户对Table的CRUD操作
2. 记录Region在哪台HRegionServer上
3. 在Region Split后，负责新Region的分配
4. 新机器加入时，管理HRegionServer的负载均衡，调整Region分布
5. 在HRegionServer宕机后，负责失效HRegionServer上的Regions迁移

## 6.7 HRegionServer的职责

1. HRegionServer主要负责响应用户的IO请求，向HDFS文件系统中读写数据，是HBase中最核心的模块
2. HRegionServer管理了很多table的分区，也就是Region

## 6.8 Client的职责

1. HBase Client 使用HBase的RPC机制与HMaster和HRegionServer进行通信
    - 管理类操作： Client与HMaster进行RPC
    - 数据读写类操作：Client与HRegionServer进行RPC

## 6.9 HBase读写原理图

![](15111658079801.jpg)

# 7 HBase开发

## 7.2 基本API使用


```java
package hbase.base_api;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.*;
import org.apache.hadoop.hbase.util.Bytes;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author NikoBelic
 * @create 2017/11/20 17:49
 */
public class HbaseApiTest
{
    static Configuration conf = null;
    static HBaseAdmin admin = null;
    static Connection connection = null;
    static Table table = null;

    @Before
    public void init() throws IOException
    {
        // 创建配置类
        conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "127.0.0.1");
        conf.set("hbase.zookeeper.property.clientPort", "2181");
        // 创建表管理类
        admin = new HBaseAdmin(conf);
        // 创建连接并获取表
        connection = ConnectionFactory.createConnection(conf);
        table = connection.getTable(TableName.valueOf("test"));
    }


    @Test
    public void testCreateTable() throws IOException
    {
        // 创建表描述类
        HTableDescriptor tableDesc = new HTableDescriptor("test");
        // 创建列族描述符类
        HColumnDescriptor myCf1 = new HColumnDescriptor("my_cf1");
        HColumnDescriptor myCf2 = new HColumnDescriptor("my_cf2");
        // 创建表操作(一般用shell，很少用代码)
        tableDesc.addFamily(myCf1);
        tableDesc.addFamily(myCf2);
        admin.createTable(tableDesc);
    }

    @Test
    public void testDeleteTable() throws IOException
    {
        admin.disableTable("test");
        admin.deleteTable("test");
    }

    @Test
    public void testInsertSingleRecord() throws IOException
    {
        Put rk1 = new Put(Bytes.toBytes("rk1"));
        rk1.addColumn(Bytes.toBytes("my_cf1"), Bytes.toBytes("name"), Bytes.toBytes("NikoBelic"));
        table.put(rk1);
    }

    @Test
    public void testInsertMultipleRecords() throws IOException
    {
        List<Put> rowKeyList = new ArrayList<>();
        for (int i = 0; i < 10; i++)
        {
            Put rk = new Put(Bytes.toBytes(i));
            rk.addColumn(Bytes.toBytes("my_cf1"), Bytes.toBytes("name"), Bytes.toBytes("Name" + i));
            rk.addColumn(Bytes.toBytes("my_cf2"), Bytes.toBytes("age"), Bytes.toBytes("Age" + i));
            rowKeyList.add(rk);
        }
        table.put(rowKeyList);
    }

    @Test
    public void testDeleteRecord() throws IOException
    {
        Delete delete = new Delete(Bytes.toBytes(5));
        table.delete(delete);
    }

    @Test
    public void testGetSingleRecord() throws IOException
    {
        Get get = new Get(Bytes.toBytes(4));
        Result result = table.get(get);
        printRow(result);
    }

    @Test
    public void testGetMultipleRecords() throws IOException
    {
        Scan scan = new Scan();
        // 这里的Row设置的是RowKey！而不是行号
        scan.setStartRow(Bytes.toBytes(1));
        scan.setStopRow(Bytes.toBytes(10));
        //scan.setStartRow(Bytes.toBytes("Shit"));
        //scan.setStopRow(Bytes.toBytes("Shit2"));
        ResultScanner scanner = table.getScanner(scan);
        printResultScanner(scanner);
    }

    @Test
    public void testFilter() throws IOException
    {
        /*
        FilterList 代表一个过滤器列表，可以添加多个过滤器进行查询，多个过滤器之间的关系有：
        与关系（符合所有）：FilterList.Operator.MUST_PASS_ALL
        或关系（符合任一）：FilterList.Operator.MUST_PASS_ONE

         过滤器的种类：
            SingleColumnValueFilter - 列值过滤器
                - 过滤列值的相等、不等、范围等
                - 注意：如果过滤器过滤的列在数据表中有的行中不存在，那么这个过滤器对此行无法过滤。

            ColumnPrefixFilter - 列名前缀过滤器
                - 过滤指定前缀的列名

            MultipleColumnPrefixFilter - 多个列名前缀过滤器
                - 过滤多个指定前缀的列名

            RowFilter - rowKey过滤器
                - 通过正则，过滤rowKey值。
                - 通常根据rowkey来指定范围时，使用scan扫描器的StartRow和StopRow方法比较好。

         */
        FilterList filterList = new FilterList(FilterList.Operator.MUST_PASS_ALL);
        Scan s1 = new Scan();

        // 1. 列值过滤器
        SingleColumnValueFilter singleColumnValueFilter = new SingleColumnValueFilter(
                Bytes.toBytes("my_cf1"), Bytes.toBytes("name"),
                CompareFilter.CompareOp.EQUAL,
                Bytes.toBytes("Name1"));

        // 2. 列名前缀过滤器
        ColumnPrefixFilter columnPrefixFilter = new ColumnPrefixFilter(Bytes.toBytes("ad"));

        // 3. 多个列值前缀过滤器
        byte[][] prefixes = new byte[][]{Bytes.toBytes("na"), Bytes.toBytes("ad")};
        MultipleColumnPrefixFilter multipleColumnPrefixFilter = new MultipleColumnPrefixFilter(prefixes);

        // 4. RowKey过滤器
        RowFilter rowFilter = new RowFilter(CompareFilter.CompareOp.EQUAL, new RegexStringComparator("^Shit"));

        // 若设置，则只返回指定的cell，同一行中的其他cell不返回
        //s1.addColumn(Bytes.toBytes("my_cf1"), Bytes.toBytes("name"));
        //s1.addColumn(Bytes.toBytes("my_cf2"), Bytes.toBytes("age"));

        // 设置过滤器
        //filterList.addFilter(singleColumnValueFilter);
        //filterList.addFilter(columnPrefixFilter);
        //filterList.addFilter(multipleColumnPrefixFilter);
        filterList.addFilter(rowFilter);
        s1.setFilter(filterList);
        ResultScanner scanner = table.getScanner(s1);
        printResultScanner(scanner);
    }

    public void printResultScanner(ResultScanner scanner)
    {
        for (Result row : scanner)
        {
            // Hbase中一个RowKey会包含[1,n]条记录，所以需要循环
            System.out.println("\n RowKey:" + String.valueOf(Bytes.toInt(row.getRow())));
            printRow(row);
        }
    }

    public void printRow(Result row)
    {
        for (Cell cell : row.rawCells())
        {
            System.out.print(String.valueOf(Bytes.toInt(cell.getRow())) + " ");
            System.out.print(new String(cell.getFamily()) + ":");
            System.out.print(new String(cell.getQualifier()) + " = ");
            System.out.print(new String(cell.getValue()));
            System.out.print("   Timestamp = " + cell.getTimestamp());
            System.out.println("");
        }
    }

}

```

## 7.3 MapReduce操作HBase



```java
package hbase.mr;

import org.apache.commons.io.FileUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
import org.apache.hadoop.hbase.mapreduce.TableMapper;
import org.apache.hadoop.hbase.mapreduce.TableReducer;
import org.apache.hadoop.hbase.util.Bytes;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;

import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author NikoBelic
 * @create 2017/11/20 20:12
 */
public class HBaseMr
{
    static Configuration config = null;

    static
    {
        config = HBaseConfiguration.create();
        config.set("hbase.zookeeper.quorum", "127.0.0.1");
        config.set("hbase.zookeeper.property.clientPort", "2181");
    }

    public static final String sourceTableName = "word";
    public static final String colf = "content";
    public static final String col = "info";
    public static final String targetTableName = "stat";

    public static void initTB()
    {
        HTable table;
        HBaseAdmin admin;
        try
        {
            admin = new HBaseAdmin(config);
            // 删除表
            if (admin.tableExists(sourceTableName) || admin.tableExists(targetTableName))
            {
                System.out.println("表已经存在了, 正在删除重建");
                admin.disableTable(sourceTableName);
                admin.deleteTable(sourceTableName);
                admin.disableTable(targetTableName);
                admin.deleteTable(targetTableName);
            }
            // 创建表
            HTableDescriptor sourceTableDesc = new HTableDescriptor(sourceTableName);
            HColumnDescriptor sourceTableCF = new HColumnDescriptor(colf);
            sourceTableDesc.addFamily(sourceTableCF);
            admin.createTable(sourceTableDesc);

            HTableDescriptor targetTableDesc = new HTableDescriptor(targetTableName);
            HColumnDescriptor targetTableCF = new HColumnDescriptor(colf);
            targetTableDesc.addFamily(targetTableCF);
            admin.createTable(targetTableDesc);

            // 插入数据
            table = new HTable(config, sourceTableName);
            table.setAutoFlush(false);
            table.setWriteBufferSize(5);
            List<Put> putList = new ArrayList<>();
            int lineCounter = 1;
            for (String line : FileUtils.readLines(new File("/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/data/paper.txt")))
            {
                putList.add(new Put(
                        Bytes.toBytes(String.valueOf(lineCounter++)))
                        .addColumn(Bytes.toBytes(colf), Bytes.toBytes(col), Bytes.toBytes(line)));

            }
            table.put(putList);
            table.flushCommits();
            putList.clear();
        } catch (MasterNotRunningException e)
        {
            e.printStackTrace();
        } catch (ZooKeeperConnectionException e)
        {
            e.printStackTrace();
        } catch (IOException e)
        {
            e.printStackTrace();
        }
    }

    /**
     * Mapper
     * ImmutableBytesWritable: 输入类型，表示rowkey
     * Text: 输出类型，表示单词
     * IntWritable: 输出类型，表示单词数量
     *
     * @Author SeawayLee
     * @Date 2017/11/21 17:07
     */
    public static class MyMapper extends TableMapper<Text, IntWritable>
    {
        private static IntWritable one = new IntWritable(1);
        private static Text word = new Text();

        @Override
        protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException
        {
            //获取一行数据中的colf：col       表里面只有一个列族，所以就直接获取每一行的值
            String words = Bytes.toString(value.getValue(Bytes.toBytes(colf), Bytes.toBytes(col)));
            String wordArray[] = words.split(" ");
            for (String str : wordArray)
            {
                word.set(str);
                context.write(word, one);
            }
        }
    }

    /**
     * Reducer
     * Text: 输入的key类型
     * IntWritable: 输入的value类型
     * ImmutableBytesWritable: 输出类型，表示rowkey的类型
     *
     * @Author SeawayLee
     * @Date 2017/11/21 17:05
     */
    public static class MyReducer extends TableReducer<Text, IntWritable, ImmutableBytesWritable>
    {
        @Override
        protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException
        {
            int sum = 0;
            for (IntWritable value : values)
            {
                sum += value.get();
            }
            Put put = new Put(Bytes.toBytes(key.toString()));
            put.addColumn(Bytes.toBytes(colf), Bytes.toBytes(col), Bytes.toBytes(String.valueOf(sum)));
            context.write(new ImmutableBytesWritable(Bytes.toBytes(key.toString())), put);
        }
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException
    {
        // 初始化表
        initTB();
        Job job = new Job(config, "HBaseMr");
        job.setJarByClass(HBaseMr.class);
        Scan scan = new Scan();
        scan.addColumn(Bytes.toBytes(colf), Bytes.toBytes(col));
        // 配置Mapper、Reducer
        TableMapReduceUtil.initTableMapperJob(sourceTableName, scan, MyMapper.class, Text.class, IntWritable.class, job);
        TableMapReduceUtil.initTableReducerJob(targetTableName, MyReducer.class, job);
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }


}

```


**写入HbaseTable的文章内容**
![](15112560067823.jpg)

**统计结果**

![](15112560297597.jpg)







