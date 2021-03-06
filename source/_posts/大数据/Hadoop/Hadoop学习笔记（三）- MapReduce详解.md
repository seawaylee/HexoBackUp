---
title: Hadoop学习笔记（三）- MapReduce详解
date: 2017-05-12 13:55:17
tags: [大数据,Hadoop]
---


# 1 MapReduce背景及原理

## 1.1 背景
Mapreduce是一个**分布式运算程序的编程框架**，是用户开发“基于hadoop的数据分析应用”的核心框架；
Mapreduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个hadoop集群上；

**Why MapReduce?**

* （1）海量数据在单机上处理因为硬件资源限制，无法胜任
* （2）而一旦将单机版程序扩展到集群来分布式运行，将极大增加程序的复杂度和开发难度
* （3）引入mapreduce框架后，开发人员可以将绝大部分工作集中在业务逻辑的开发上，而将分布式计算中的复杂性交由框架来处理


<!--more-->

## 1.2 MR框架结构核心及运行机制


### 1.2.1 结构

一个完整的mapreduce程序在分布式运行时有三类实例进程：

* 1、MRAppMaster：负责整个程序的过程调度及状态协调
* 2、mapTask：负责map阶段的整个数据处理流程
* 3、ReduceTask：负责reduce阶段的整个数据处理流程

![](mapreduce运行全流程.png)

**流程解析**

1. 一个mr程序启动的时候，最先启动的是MRAppMaster，MRAppMaster启动后根据本次job的描述信息，计算出需要的maptask实例数量，然后向集群申请机器启动相应数量的maptask进程

2. maptask进程启动之后，根据给定的数据切片范围进行数据处理，主体流程为：
    * 利用客户指定的inputformat来获取RecordReader读取数据，形成输入KV对
    * 将输入KV对传递给客户定义的map()方法，做逻辑运算，并将map()方法输出的KV对收集到缓存
    * 将缓存中的KV对按照K分区排序后不断溢写到磁盘文件

3. MRAppMaster监控到所有maptask进程任务完成之后，会根据客户指定的参数启动相应数量的reducetask进程，并告知reducetask进程要处理的数据范围（数据分区）

4. Reducetask进程启动之后，根据MRAppMaster告知的待处理数据所在位置，从若干台maptask运行所在机器上获取到若干个maptask输出结果文件，并在本地进行重新归并排序，然后按照相同key的KV为一个组，调用客户定义的reduce()方法进行逻辑运算，并收集运算输出的结果KV，然后调用客户指定的outputformat将结果数据输出到外部存储

### 1.2.2 MapTask并行度的决定机制

一个job的map阶段并行度由客户端在提交job时决定，而客户端对map阶段并行度的规划的基本逻辑为：

将待处理数据执行逻辑切片（即按照一个特定切片大小，将待处理数据划分成逻辑上的多个split），然后**每一个split分配一个mapTask并行实例处理**
这段逻辑及形成的切片规划描述文件，由FileInputFormat实现类的getSplits()方法完成，其过程如下图：

![](切片2.png)

![](切片示意.png)


### 1.2.3 FileInputFormat切片机制

**切片定义在InputFormat类中的getSplit()方法**

**FileInputFormat中默认的切片机制**

* 简单地按照文件的内容长度进行切片
* 切片大小，默认等于block大小
* 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

比如待处理数据有两个文件：
file1.txt    320M
file2.txt    10M

经过FileInputFormat的切片机制运算后，形成的切片信息如下：  
file1.txt.split1--  0~128
file1.txt.split2--  128~256
file1.txt.split3--  256~320
file2.txt.split1--  0~10M

**FileIputFormat中切片大小的参数配置**

通过分析源码，在FileInputFormat中，计算切片大小的逻辑：Math.max(minSize, Math.min(maxSize, blockSize));  切片主要由这几个值来运算决定


```
minsize：默认值：1  
配置参数： mapreduce.input.fileinputformat.split.minsize    
maxsize：默认值：Long.MAXValue  
配置参数：mapreduce.input.fileinputformat.split.maxsize
```

**因此，默认情况下，切片大小=blocksize**

maxsize（切片最大值）：
参数如果调得比blocksize小，则会让切片变小，而且就等于配置的这个参数的值
minsize （切片最小值）：
参数调的比blockSize大，则可以让切片变得比blocksize还大

选择并发数的影响因素：

* 运算节点的硬件配置
* 运算任务的类型：CPU密集型还是IO密集型
* 运算任务的数据量


### 1.2.4 ReduceTask并行度的决定
reducetask的并行度同样影响整个job的执行并发度和执行效率，但**与maptask的并发数由切片数决定不同，Reducetask数量的决定是可以直接手动设置：**

//默认值是1，手动设置为4
job.setNumReduceTasks(4);

如果数据分布不均匀，就有可能在reduce阶段产生**数据倾斜**
注意： reducetask数量并不是任意设置，还要考虑业务逻辑需求，有些情况下，需要计算**全局汇总结果，就只能有1个reducetask**

尽量不要运行太多的reduce task。对大多数job来说，最好reduce的个数最多和集群中的reduce持平，或者比集群的 reduce slots小。这个对于小集群而言，尤其重要。


## 1.3 MR程序运行模式

### 1.3.1 本地运行模式

1. mapreduce程序是被提交给LocalJobRunner在本地以单进程的形式运行
2. 而处理的数据及输出结果可以在本地文件系统，也可以在hdfs上
3. 怎样实现本地运行？写一个程序，不要带集群的配置文件（本质是你的mr程序的conf中是否有mapreduce.framework.name=local以及yarn.resourcemanager.hostname参数）
4. 本地模式非常便于进行业务逻辑的debug，只要在eclipse中打断点即可

### 1.3.2 集群运行模式

1. 将mapreduce程序提交给yarn集群resourcemanager，分发到很多的节点上并发执行
2. 处理的数据和输出结果应该位于hdfs文件系统
3. 提交集群的实现步骤：
    * A、将程序打成JAR包，然后在集群的任意一个节点上用hadoop命令启动
    `$ hadoop jar wordcount.jar cn.itcast.bigdata.mrsimple.WordCountDriver inputpath outputpath`
    * B、直接在linux的eclipse中运行main方法（项目中要带参数：mapreduce.framework.name=yarn以及yarn的两个基本配置）
    * C、如果要在windows的eclipse中提交job给集群，则要修改YarnRunner类

## 1.4 Combiner

1. combiner是MR程序中Mapper和Reducer之外的一种组件
2. combiner组件的父类就是Reducer
3. combiner和reducer的区别在于运行的位置：
    * Combiner是在每一个maptask所在的节点运行
    * Reducer是接收全局所有Mapper的输出结果；
4. combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量
具体实现步骤：
    1. 自定义一个combiner继承Reducer，重写reduce方法
    2. 在job中设置：  job.setCombinerClass(CustomCombiner.class)
5. combiner能够应用的前提是不能影响最终的业务逻辑。而且，combiner的输出kv应该跟reducer的输入kv类型要对应起来

## 1.5 Partitioner

Mapreduce中会将mapTask输出的kv对，按照相同key分组，然后分发给不同的reduceTask
默认的分发规则为：根据key的hashcode%reducetask数来分发
组件Partitioner能够改写数据分发（分组）

## 1.6 GroupingComparator

自定义GroupingComparator可以将Map输出的K-V按照自己的需求进行聚合

# 2 MR 编程实践

## 2.1 编程规范

1. 用户编写的程序分成三个部分：Mapper，Reducer，Driver(提交运行mr程序的客户端)
2. Mapper的输入数据是KV对的形式（KV的类型可自定义）
3. Mapper的输出数据是KV对的形式（KV的类型可自定义）
4. Mapper中的业务逻辑写在map()方法中
5. map()方法（maptask进程）对每一个<K,V>调用一次
6. Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
7. Reducer的业务逻辑写在reduce()方法中
8. ReduceTask进程对每一组相同k的<k,v>组调用一次reduce()方法
9. 用户自定义的Mapper和Reducer都要继承各自的父类
10. 整个程序需要一个Drvier来进行提交，提交的是一个描述了各种必要信息的job对象

## 2.2 WordCount示例代码

**Mapper**


```java
/**
 * KEYIN: 默认情况下，是MR框架锁读到的一行文本的起始偏移量，Long
 * VALUEIN: 默认情况下，是MR框架所读到一行文本的内容，String
 * KEYOUT: 是用户自定义逻辑处理完成后输出数据中的Key，在此处是单词，String
 * VALUEOUT: 是用户自定义逻辑处理完成后输出数据中的Value，在此处是单词次数，Integer
 *
 * 在Hadoop中有更精简的序列化接口（纯粹的数据），所以不直接用Java中的基本数据类型对象（虽然支持序列化，但是有很多冗余信息例如继承结构），而用LongWritable
 * @author NikoBelic
 * @create 2017/5/3 09:16
 */
public class WordCountMapper extends Mapper<LongWritable,Text,Text,IntWritable>
{
    /**
     * Map阶段的业务逻辑
     * MapTask会对每一行输入数据调用一次此方法
     * @Author SeawayLee
     * @Date 2017/05/03 09:29
     */
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        // 将文本内容转换成字符串
        String line = value.toString();
        // 对一行文本切分成单词数组
        String[] words = line.split(" ");
        // 将单词输出为<word,1>
        for (String word : words)
        {
            // 将单词作为key，将次数1作为value，以便于后续的数据分发
            // 可以根据单词分发，以便于相同的单词会到相同的ReduceTask中
            context.write(new Text(word), new IntWritable(1));
        }
        System.out.println("MapTask正在处理第 " + key.toString() + " 行文本,ObjHashCode:" + this.hashCode() + ",IP:"  + Inet4Address.getLocalHost().getHostAddress());
    }
}

```

**Reducer**


```java
/**
 * KEYIN,VALUEIN 对应 Mapper输出的KEYOUT,VALUEOUT
 * KEYOUT,VALUEOUT 是自动以Reduce逻辑处理结果的输出数据类型
 * KEYOUT是单词，VALUEOUT是总次数
 * @author NikoBelic
 * @create 2017/5/3 09:49
 */
public class WordCountReducer extends Reducer<Text,IntWritable,Text,IntWritable>
{
    /**
     * ReduceTask处理逻辑
     * <Niko,1> <Niko,1> <Niko,1> <Niko,1>
     * <Belic,1> <Belic,1> <Belic,1> <Belic,1>
     * @param key 一组相同单词kv的key
     * @param values
     *
     * @Author SeawayLee
     * @Date 2017/05/03 09:52
     */
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException
    {
        int count = 0;
        for (IntWritable value : values)
        {
            count += value.get();
        }

        context.write(key, new IntWritable(count));
        System.out.println("ReduceTask正在处理第 " + key.toString() + " 行文本,ObjHashCode:" + this.hashCode() + ",IP:"  + Inet4Address.getLocalHost().getHostAddress());
    }
}

```


**Driver**


```java
/**
 * Driver相当于Yarn集群的客户端
 * 需要在此封装MR程序的相关运行参数，指定jar包
 * 最后提交给Yarn
 *
 * @author NikoBelic
 * @create 2017/5/3
 * 10:00
 */
public class WordCountDriver
{
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException, URISyntaxException
    {
        Configuration conf = new Configuration();
        conf.set("mapreduce.framework.name", "yarn");
        conf.set("yarn.resourcemanager.hostname", "10.5.151.241");
        //conf.set("mapred.jar", "hadoop.jar");
        Job job = Job.getInstance(conf);

        //job.setJar("/home/hadoop/wc.jar");
        // 指定本程序的jar包所在的本地路径，需要提交给Yarn
        // setJar方法传入jar包绝对路径，必须将jar部署到指定位置，很不灵活
        // 而使用setJarByClass，可以从JVM中找到该类，比较常用
        job.setJarByClass(WordCountDriver.class);
        job.setJar("hadoop.jar");

        // 指定本业务job要使用的Map和Reduce业务类
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        // 指定Mapper输数据的KV类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // 指定最终输出数据的KV类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 指定job的输入源
        FileInputFormat.setInputPaths(job, new Path(args[0]));

        // 指定job的输出结构存储位置
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop1:9000"), conf, "root");
        if (fs.exists(new Path(args[1])))
        {
            fs.delete(new Path(args[1]), true);
        }
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 将job中配置的相关参数，以及job所用的java类所在的jar包提交给yarn去运行
        //job.submit();
        // 以阻塞的方式提交job，便于查看job处理进度
        boolean res = job.waitForCompletion(true);
        System.exit(res ? 0 : 1);
    }
}

```

# 3 MapReduce参数优化

## 3.1 资源相关参数

**以下参数是在用户自己的mr应用程序中配置就可以生效**

* (1) mapreduce.map.memory.mb: 一个Map Task可使用的资源上限（单位:MB），默认为1024。如果Map Task实际使用的资源量超过该值，则会被强制杀死。
* (2) mapreduce.reduce.memory.mb: 一个Reduce Task可使用的资源上限（单位:MB），默认为1024。如果Reduce Task实际使用的资源量超过该值，则会被强制杀死。
* (3) mapreduce.map.java.opts: Map Task的JVM参数，你可以在此配置默认的java heap size等参数, e.g.
* “-Xmx1024m -verbose:gc -Xloggc:/tmp/@taskid@.gc” （@taskid@会被Hadoop框架自动换为相应的taskid）, 默认值: “”
* (4) mapreduce.reduce.java.opts: Reduce Task的JVM参数，你可以在此配置默认的java heap size等参数, e.g.
* “-Xmx1024m -verbose:gc -Xloggc:/tmp/@taskid@.gc”, 默认值: “”
* (5) mapreduce.map.cpu.vcores: 每个Map task可使用的最多cpu core数目, 默认值: 1
* (6) mapreduce.reduce.cpu.vcores: 每个Reduce task可使用的最多cpu core数目, 默认值: 1

**以下在yarn启动之前就配置在服务器的配置文件中才能生效**

* (7) yarn.scheduler.minimum-allocation-mb	  1024   给应用程序container分配的最小内存
* (8) yarn.scheduler.maximum-allocation-mb	  8192	给应用程序container分配的最大内存
* (9) yarn.scheduler.minimum-allocation-vcores	1	
* (10)yarn.scheduler.maximum-allocation-vcores	32
* (11)yarn.nodemanager.resource.memory-mb   8192  

**shuffle性能优化的关键参数，应在yarn启动之前就配置好**

mapreduce.task.io.sort.mb   100         //shuffle的环形缓冲区大小，默认100m
mapreduce.map.sort.spill.percent   0.8    //环形缓冲区溢出的阈值，默认80%

## 3.2 容错相关参数

* (1) mapreduce.map.maxattempts: 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。
* (2) mapreduce.reduce.maxattempts: 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。
* (3) mapreduce.map.failures.maxpercent: 当失败的Map Task失败比例超过该值为，整个作业则失败，默认值为0. 如果你的应用程序允许丢弃部分输入数据，则该该值设为一个大于0的值，比如5，表示如果有低于5%的Map Task失败（如果一个Map Task重试次数超过mapreduce.map.maxattempts，则认为这个Map Task失败，其对应的输入数据将不会产生任何结果），整个作业扔认为成功。
* (4) mapreduce.reduce.failures.maxpercent: 当失败的Reduce Task失败比例超过该值为，整个作业则失败，默认值为0.
* (5) mapreduce.task.timeout: Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该task处于block状态，可能是卡住了，也许永远会卡主，为了防止因为用户程序永远block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是300000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。

## 3.3 本地运行mapreduce 作业

设置以下几个参数:
mapreduce.framework.name=local
mapreduce.jobtracker.address=local
fs.defaultFS=local

## 3.4 效率和稳定性相关参数

* (1) mapreduce.map.speculative: 是否为Map Task打开推测执行机制，默认为false
* (2) mapreduce.reduce.speculative: 是否为Reduce Task打开推测执行机制，默认为false
* (3) mapreduce.job.user.classpath.first & mapreduce.task.classpath.user.precedence：当同一个class同时出现在用户jar包和hadoop jar中时，优先使用哪个jar包中的class，默认为false，表示优先使用hadoop jar中的class。
* (4) mapreduce.input.fileinputformat.split.minsize: FileInputFormat做切片时的最小切片大小，(5)mapreduce.input.fileinputformat.split.maxsize:  FileInputFormat做切片时的最大切片大小(切片的默认大小就等于blocksize，即 134217728)


# 4 MR编程小结

MR在编程的时候，基本上一个固化的模式，没有太多可灵活改变的地方，除了以下几处：

- 1、输入数据接口：InputFormat —> FileInputFormat(文件类型数据读取的通用抽象类)  DBInputFormat （数据库数据读取的通用抽象类） 默认使用的实现类是： TextInputFormat  `job.setInputFormatClass(TextInputFormat.class)`

    TextInputFormat的功能逻辑是：一次**读一行**文本，然后将该行的起始偏移量作为key，行内容作为value返回
   
- 2、逻辑处理接口： Mapper  
   完全需要用户自己去实现其中  map()   setup()   clean()   
   
- 3、Map输出的结果在shuffle阶段会被partition以及sort，此处有两个接口可自定义：
    - **Partitioner**
    
		有默认实现 HashPartitioner，逻辑是  根据key和numReduces来返回一个分区号； key.hashCode()&Integer.MAXVALUE % numReduces
	通常情况下，用默认的这个HashPartitioner就可以，如果业务上有特别的需求，可以自定义
    - **Comparable**

		当我们用自定义的对象作为key来输出时，就必须要实现WritableComparable接口，override其中的compareTo()方法

- 4、Reduce端的数据分组比较接口 ： **Groupingcomparator**
    reduceTask拿到输入数据（一个partition的**所有**数据）后，首先需要对数据进行**分组**，其分组的**默认原则是key相同**，然后对每一组kv数据调用一次reduce()方法，并且将这一组kv中的**第一个**kv的key作为参数传给reduce的key，将这一组数据的value的迭代器传给reduce()的values参数
	
    利用上述这个机制，我们可以实现一个**高效的分组取最大值的逻辑**：

    - 自定义一个bean对象用来封装我们的数据，然后改写其compareTo方法产生倒序排序的效果
    - 然后自定义一个**Groupingcomparator**，将bean对象的分组逻辑改成按照我们的业务分组id来分组（比如订单号）
	这样，我们要取的最大值就是reduce()方法中传进来key
	

- 5、逻辑处理接口：Reducer
	完全需要用户自己去实现其中  reduce()   setup()   clean()   

- 6、输出数据接口： OutputFormat  ---> 有一系列子类  FileOutputformat  DBoutputFormat  .....
	默认实现类是TextOutputFormat，功能逻辑是：  将每一个KV对向目标文本文件中**输出为一行**
	
	
	
	

