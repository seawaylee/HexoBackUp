---
title: Hadoop学习笔记（五）- MapReduce实践
date: 2017-06-12 22:52:45
tags: [大数据,Hadoop]
---


在生产环境中，其实很少自己去写MR程序，一般都是直接在Hive上写SQL完成业务逻辑，但动手写MR程序有助于我们理解MR原理，而不是一个只会写SQL的所谓的“数据分析师”。: )

## 1 流量分析排序

**需求：** 对日志数据中的上下行流量信息汇总，并输出按照总流量倒序排序的结果。


```commandline
1363157985066 	13726230503	00-FD-07-A4-72-B8:CMCC	120.196.100.82	i02.c.aliimg.com		24	27	2481	24681	200
1363157995052 	13826544101	5C-0E-8B-C7-F1-E0:CMCC	120.197.40.4			4	0	264	0	200
1363157991076 	13926435656	20-10-7A-28-CC-0A:CMCC	120.196.100.99			2	4	132	1512	200
1363154400022 	13926251106	5C-0E-8B-8B-B1-50:CMCC	120.197.40.4			4	0	240	0	200
1363157993044 	18211575961	94-71-AC-CD-E6-18:CMCC-EASY	120.196.100.99	iface.qiyi.com	视频网站	15	12	1527	2106	200
1363157995074 	84138413	5C-0E-8B-8C-E8-20:7DaysInn	120.197.40.4	122.72.52.12		20	16	4116	1432	200
1363157993055 	13560439658	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			18	15	1116	954	200
1363157995033 	15920133257	5C-0E-8B-C7-BA-20:CMCC	120.197.40.4	sug.so.360.cn	信息安全	20	20	3156	2936	200
1363157983019 	13719199419	68-A1-B7-03-07-B1:CMCC-EASY	120.196.100.82			4	0	240	0	200
1363157984041 	13660577991	5C-0E-8B-92-5C-20:CMCC-EASY	120.197.40.4	s19.cnzz.com	站点统计	24	9	6960	690	200
1363157973098 	15013685858	5C-0E-8B-C7-F7-90:CMCC	120.197.40.4	rank.ie.sogou.com	搜索引擎	28	27	3659	3538	200
1363157986029 	15989002119	E8-99-C4-4E-93-E0:CMCC-EASY	120.196.100.99	www.umeng.com	站点统计	3	3	1938	180	200
1363157992093 	13560439658	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			15	9	918	4938	200
1363157986041 	13480253104	5C-0E-8B-C7-FC-80:CMCC-EASY	120.197.40.4			3	3	180	180	200
1363157984040 	13602846565	5C-0E-8B-8B-B6-00:CMCC	120.197.40.4	2052.flash2-http.qq.com	综合门户	15	12	1938	2910	200
1363157995093 	13922314466	00-FD-07-A2-EC-BA:CMCC	120.196.100.82	img.qfc.cn		12	12	3008	3720	200
1363157982040 	13502468823	5C-0A-5B-6A-0B-D4:CMCC-EASY	120.196.100.99	y0.ifengimg.com	综合门户	57	102	7335	110349	200
1363157986072 	18320173382	84-25-DB-4F-10-1A:CMCC-EASY	120.196.100.99	input.shouji.sogou.com	搜索引擎	21	18	9531	2412	200
1363157990043 	13925057413	00-1F-64-E1-E6-9A:CMCC	120.196.100.55	t3.baidu.com	搜索引擎	69	63	11058	48243	200
1363157988072 	13760778710	00-FD-07-A4-7B-08:CMCC	120.196.100.82			2	2	120	120	200
1363157985066 	13726238888	00-FD-07-A4-72-B8:CMCC	120.196.100.82	i02.c.aliimg.com		24	27	2481	24681	200
1363157993055 	13560436666	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			18	15	1116	954	200
```

**分析：**
基本思路：
Job1：Map读取文件后输出 <phoneNumber,flowbean>  Reduce计算flowSum 输出到文件
Job2：Map输出<FlowBean,Text>,Map->Reduce之间的shuffle会帮助我们对flowbean进行排序，Reduce不用作任何操作。

实现自定义的bean来封装流量信息，并将bean作为map输出的key来传输。

MR程序在处理数据的过程中会对数据排序(map输出的kv对传输到reduce之前，会排序)，排序的依据是map输出的key。

所以，我们如果要实现自己需要的排序规则，则可以考虑将排序因素放到key中，让key实现接口：WritableComparable，然后重写key的compareTo方法

<!--more-->

**代码实现**

**实现序列化的DO对象**

```java
package hadoop.mr.flowcount;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * @author NikoBelic
 * @create 2017/5/7 22:50
 */
public class FlowBean implements WritableComparable<FlowBean>
{
    private Long upFlow;
    private Long downFlow;
    private Long sumFlow;

    public FlowBean()
    {
    }

    public FlowBean(Long upFlow, Long downFlow)
    {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.sumFlow = this.upFlow + this.downFlow;
    }

    public Long getUpFlow()
    {
        return upFlow;
    }

    public void setUpFlow(Long upFlow)
    {
        this.upFlow = upFlow;
    }

    public Long getDownFlow()
    {
        return downFlow;
    }

    public void setDownFlow(Long downFlow)
    {
        this.downFlow = downFlow;
    }

    public Long getSumFlow()
    {
        return sumFlow;
    }

    public void setSumFlow(Long sumFlow)
    {
        this.sumFlow = sumFlow;
    }

    public void set(Long upFlow, Long downFlow,Long sumFlow)
    {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.sumFlow = sumFlow;
    }
    @Override
    public String toString()
    {
        return upFlow + " " + downFlow + " " + sumFlow;
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException
    {
        dataOutput.writeLong(upFlow);
        dataOutput.writeLong(downFlow);
        dataOutput.writeLong(sumFlow);
    }

    @Override
    public void readFields(DataInput dataInput) throws IOException
    {
        this.upFlow = dataInput.readLong();
        this.downFlow  = dataInput.readLong();
        this.sumFlow = dataInput.readLong();
    }

    @Override
    public int compareTo(FlowBean o)
    {
        return this.sumFlow > o.getSumFlow() ? -1 : 1;
    }
}

```


**计算FlowSumMapReduce**


```java
package hadoop.mr.flowcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.Iterator;

/**
 * @author NikoBelic
 * @create 2017/5/7 22:54
 */
public class FlowCount
{
    public static void main(String[] args) throws IOException, URISyntaxException, InterruptedException, ClassNotFoundException
    {
        String inputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/flow.txt";
        //String inputPath = "hdfs://hadoop1:9000/flowbean/flow.txt";
        String outputPath = "hdfs://localhost:9000/flowbean/result";
        Configuration conf = new Configuration();
        //conf.set("mapreduce.framework.name", "yarn");
        //conf.set("yarn.resourcemanager.hostname", "hadoop1");

        Job job = Job.getInstance(conf );
        job.setUser("root");
        job.setJarByClass(FlowCount.class);

        job.setMapperClass(FlowCountMapper.class);
        job.setReducerClass(FlowCountReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //job.setPartitionerClass(FlowCountPartitioner.class);
        //job.setNumReduceTasks(4);

        FileInputFormat.setInputPaths(job, new Path(inputPath));

        FileSystem fs = FileSystem.get(new URI(outputPath), conf, "NikoBelic");
        if (fs.exists(new Path(outputPath)))
        {
            fs.delete(new Path(outputPath));
        }
        FileOutputFormat.setOutputPath(job, new Path(outputPath));

        job.waitForCompletion(true);
    }
}

class FlowCountMapper extends Mapper<LongWritable, Text, Text, FlowBean>
{

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String[] values = value.toString().split("\\t");
        String phoneNumber = values[1];
        Long upFlow = Long.parseLong(values[values.length - 3]);
        Long downFlow = Long.parseLong(values[values.length - 2]);
        FlowBean flowBean = new FlowBean(upFlow, downFlow);
        context.write(new Text(phoneNumber), flowBean);
    }
}

class FlowCountReducer extends Reducer<Text, FlowBean, Text, FlowBean>
{
    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException
    {
        Iterator<FlowBean> iterator = values.iterator();
        FlowBean currBean;
        Long totalUp = 0L;
        Long totalDown = 0L;
        while (iterator.hasNext())
        {
            currBean = iterator.next();
            totalUp += currBean.getUpFlow();
            totalDown += currBean.getDownFlow();
        }
        context.write(key, new FlowBean(totalUp, totalDown));
    }
}

class FlowCountPartitioner extends Partitioner<Text, FlowBean>
{

    @Override
    public int getPartition(Text key, FlowBean value, int i)
    {
        return key.charAt(3) % 4;
    }
}
```

**输出Sum计算结果**

```
13480253104	180 180 360
13502468823	7335 110349 117684
13560436666	1116 954 2070
13560439658	2034 5892 7926
13602846565	1938 2910 4848
13660577991	6960 690 7650
13719199419	240 0 240
13726230503	2481 24681 27162
13726238888	2481 24681 27162
13760778710	120 120 240
13826544101	264 0 264
13922314466	3008 3720 6728
13925057413	11058 48243 59301
13926251106	240 0 240
13926435656	132 1512 1644
15013685858	3659 3538 7197
15920133257	3156 2936 6092
15989002119	1938 180 2118
18211575961	1527 2106 3633
18320173382	9531 2412 11943
84138413	4116 1432 5548

```

**排序MapReduce**


```java
package hadoop.mr.flowcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

/**
 * @author NikoBelic
 * @create 2017/5/15 21:37
 */
public class FlowCountSort
{
    public static void main(String[] args) throws IOException, URISyntaxException, InterruptedException, ClassNotFoundException
    {
        String inputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/flowsum.txt";
        String outputPath = "hdfs://localhost:9000/flowsortbean/result";
        Configuration conf = new Configuration();
        //conf.set("mapreduce.framework.name", "yarn");
        //conf.set("yarn.resourcemanager.hostname", "localhost");

        Job job = Job.getInstance(conf);
        job.setJarByClass(FlowCountSort.class);

        job.setMapperClass(FlowCountSortMapper.class);
        job.setReducerClass(FlowCountSortReducer.class);

        job.setMapOutputKeyClass(FlowBean.class);
        job.setMapOutputValueClass(Text.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        //job.setPartitionerClass(FlowCountPartitioner.class);
        //job.setNumReduceTasks(4);

        FileInputFormat.setInputPaths(job, new Path(inputPath));

        FileSystem fs = FileSystem.get(new URI("hdfs://localhost:9000"), conf, "NikoBelic");
        if (fs.exists(new Path(outputPath)))
        {
            fs.delete(new Path(outputPath));
        }
        FileOutputFormat.setOutputPath(job, new Path(outputPath));

        job.waitForCompletion(true);
    }
}


class FlowCountSortMapper extends Mapper<LongWritable, Text, FlowBean, Text>
{
    private FlowBean resultBean = new FlowBean();
    private Text v = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String values[] = value.toString().split(" ");
        Long upFlow = Long.valueOf(values[1]);
        Long downFlow = Long.valueOf(values[2]);
        Long sumFlow = Long.valueOf(values[values.length - 1]);
        String phoneNumber = values[0];
        v.set(phoneNumber);
        resultBean.set(upFlow, downFlow, sumFlow);
        context.write(resultBean, v);
    }
}

class FlowCountSortReducer extends Reducer<FlowBean, Text, Text, FlowBean>
{
    @Override
    protected void reduce(FlowBean key, Iterable<Text> values, Context context) throws IOException, InterruptedException
    {
        context.write(values.iterator().next(), key);
    }
}
```

**输出排序结果**


```
13502468823	7335	110349 117684 117684
13925057413	11058	48243 59301 59301
13726238888	2481	24681 27162 27162
13726230503	2481	24681 27162 27162
18320173382	9531	2412 11943 11943
13560439658	2034	5892 7926 7926
13660577991	6960	690 7650 7650
15013685858	3659	3538 7197 7197
13922314466	3008	3720 6728 6728
15920133257	3156	2936 6092 6092
84138413	4116	1432 5548 5548
13602846565	1938	2910 4848 4848
18211575961	1527	2106 3633 3633
15989002119	1938	180 2118 2118
13560436666	1116	954 2070 2070
13926435656	132	1512 1644 1644
13480253104	180	180 360 360
13826544101	264	0 264 264
13926251106	240	0 240 240
13760778710	120	120 240 240
13719199419	240	0 240 240

```

# 2 Join算法

## 2.1 Reduce端的Join算法


**需求：**

**订单数据表t_order：**
1001,20150710,P0001,2
1002,20150710,P0001,3
1002,20150710,P0002,3
1003,20150710,P0003,3

**商品信息表t_product**
P0001,小米5,1001,2
P0002,锤子T1,1000,3
P0003,锤子,1002,4

假如数据量巨大，两表的数据是以文件的形式存储在HDFS中，需要用mapreduce程序来实现一下SQL查询运算： `select  a.id,a.date,b.name,b.category_id,b.price from t_order a join t_product b on a.pid = b.id`

**思路**

通过将关联的条件作为map输出的key，将两表满足join条件的数据并携带数据所来源的文件信息，发往同一个reduce task，在reduce中进行数据的串联。
即 map=>(productId,orderDO/productDO),reduce=>orderDO + productDO


**代码实现**

**实现序列化接口的DO**


```java
package hadoop.mr.join;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

/**
 * @author NikoBelic
 * @create 2017/5/22 21:22
 */
public class InfoBean implements Writable
{

    String orderId;
    String timestamp;
    String amount;
    String productId;
    String productName;
    String categoryId;
    String price;


    public InfoBean()
    {
    }


    public void setAll(String orderId, String timestamp, String productId, String amount, String productName, String categoryId, String price)
    {
        this.orderId = orderId;
        this.timestamp = timestamp;
        this.productId = productId;
        this.amount = amount;
        this.productName = productName;
        this.categoryId = categoryId;
        this.price = price;
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException
    {
        dataOutput.writeUTF(orderId);
        dataOutput.writeUTF(timestamp);
        dataOutput.writeUTF(productId);
        dataOutput.writeUTF(amount);
        dataOutput.writeUTF(productName);
        dataOutput.writeUTF(categoryId);
        dataOutput.writeUTF(price);
    }

    @Override
    public void readFields(DataInput dataInput) throws IOException
    {
        this.orderId = dataInput.readUTF();
        this.timestamp = dataInput.readUTF();
        this.productId = dataInput.readUTF();
        this.amount = dataInput.readUTF();
        this.productName = dataInput.readUTF();
        this.categoryId = dataInput.readUTF();
        this.price = dataInput.readUTF();
    }

    public String getOrderId()
    {
        return orderId;
    }

    public void setOrderId(String orderId)
    {
        this.orderId = orderId;
    }

    public String getTimestamp()
    {
        return timestamp;
    }

    public void setTimestamp(String timestamp)
    {
        this.timestamp = timestamp;
    }

    public String getAmount()
    {
        return amount;
    }

    public void setAmount(String amount)
    {
        this.amount = amount;
    }

    public String getProductId()
    {
        return productId;
    }

    public void setProductId(String productId)
    {
        this.productId = productId;
    }

    public String getProductName()
    {
        return productName;
    }

    public void setProductName(String productName)
    {
        this.productName = productName;
    }

    public String getCategoryId()
    {
        return categoryId;
    }

    public void setCategoryId(String categoryId)
    {
        this.categoryId = categoryId;
    }

    public String getPrice()
    {
        return price;
    }

    public void setPrice(String price)
    {
        this.price = price;
    }

    @Override
    public String toString()
    {
        return "InfoBean{" + "orderId='" + orderId + '\'' + ", timestamp='" + timestamp + '\'' + ", amount='" + amount + '\'' + ", productId='" + productId + '\'' + ", productName='" + productName + '\'' + ", categoryId='" + categoryId + '\'' + ", price='" + price + '\'' + '}';
    }
}

```

**MR**


```java
package hadoop.mr.join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URI;
import java.util.HashMap;
import java.util.Map;

/**
 * @author NikoBelic
 * @create 2017/5/24 09:55
 */
public class MapJoin
{
    public static void main(String[] args) throws Exception
    {
        String inputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/join/order.txt";
        String outputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/join/result";
        Configuration conf = new Configuration();
        //conf.set("mapreduce.framework.name", "yarn");
        //conf.set("yarn.resourcemanager.hostname", "hadoop1");

        Job job = Job.getInstance(conf);
        //job.setUser("NikoBelic");
        //job.setJarByClass(FlowCount.class);
        job.addCacheFile(new URI("/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/join/product.txt"));
        job.setUser("NikoBelicll");
        job.setMapperClass(MapperJoin.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.setInputPaths(job, new Path(inputPath));

        FileSystem fs = FileSystem.get(new URI(outputPath), conf, "NikoBelic");
        if (fs.exists(new Path(outputPath)))
        {
            fs.delete(new Path(outputPath));
        }
        FileOutputFormat.setOutputPath(job, new Path(outputPath));

        job.waitForCompletion(true);
    }
}

class MapperJoin extends Mapper<LongWritable, Text, Text, NullWritable>
{
    Map<String, String> productMap = new HashMap<>();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException
    {
        URI[] cacheFiles = context.getCacheFiles();
        for (URI cacheFile : cacheFiles)
        {
            System.out.println(cacheFile.getPath());
            BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(cacheFile.getPath())));
            String line = null;
            while ((line = br.readLine()) != null)
            {
                String[] fields = line.split(",");
                productMap.put(fields[0], fields[1]);
            }
        }
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String[] fields = value.toString().split(",");
        context.write(new Text(value.toString() + "," + productMap.get(fields[2])), NullWritable.get());
    }
}
```

**输出**


```
1001,20150710,P0001,2,小米5
1002,20150710,P0001,3,小米5
1002,20150710,P0002,3,锤子T1
1003,20150710,P0003,3,锤子
```

# 3 Map端Join算法

在上面的算法中，join的操作是在reduce阶段完成，reduce端的处理压力太大，map节点的运算负载则很低，资源利用率不高，且在reduce阶段极易产生数据倾斜。

**解决方案：**在Map端实现Map算法

**原理阐述**
适用于关联表中有小表的情形；
可以将小表分发到所有的map节点，这样，map节点就可以在本地对自己所读到的大表数据进行join并输出最终结果，可以大大提高join操作的并发度，加快处理速度

**实现示例**
--先在mapper类中预先定义好小表，进行join
--引入实际场景中的解决方案：一次加载数据库或者用distributedcache


```java
package hadoop.mr.join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URI;
import java.util.HashMap;
import java.util.Map;

/**
 * @author NikoBelic
 * @create 2017/5/24 09:55
 */
public class MapJoin
{
    public static void main(String[] args) throws Exception
    {
        String inputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/join/order.txt";
        String outputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/join/result";
        Configuration conf = new Configuration();
        //conf.set("mapreduce.framework.name", "yarn");
        //conf.set("yarn.resourcemanager.hostname", "hadoop1");

        Job job = Job.getInstance(conf);
        //job.setUser("NikoBelic");
        //job.setJarByClass(FlowCount.class);
        job.addCacheFile(new URI("/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/join/product.txt"));
        job.setUser("NikoBelicll");
        job.setMapperClass(MapperJoin.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);

        FileInputFormat.setInputPaths(job, new Path(inputPath));

        FileSystem fs = FileSystem.get(new URI(outputPath), conf, "NikoBelic");
        if (fs.exists(new Path(outputPath)))
        {
            fs.delete(new Path(outputPath));
        }
        FileOutputFormat.setOutputPath(job, new Path(outputPath));

        job.waitForCompletion(true);
    }
}

class MapperJoin extends Mapper<LongWritable, Text, Text, NullWritable>
{
    Map<String, String> productMap = new HashMap<>();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException
    {
        URI[] cacheFiles = context.getCacheFiles();
        for (URI cacheFile : cacheFiles)
        {
            System.out.println(cacheFile.getPath());
            BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(cacheFile.getPath())));
            String line = null;
            while ((line = br.readLine()) != null)
            {
                String[] fields = line.split(",");
                productMap.put(fields[0], fields[1]);
            }
        }
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String[] fields = value.toString().split(",");
        context.write(new Text(value.toString() + "," + productMap.get(fields[2])), NullWritable.get());
    }
}
```

**输出**


```
1001,20150710,P0001,2,小米5
1002,20150710,P0001,3,小米5
1002,20150710,P0002,3,锤子T1
1003,20150710,P0003,3,锤子

```


# 4 社交粉丝数据分析

**需求**

以下是qq的好友列表数据，冒号前是一个用，冒号后是该用户的所有好友（数据中的好友关系是单向的）
A:B,C,D,F,E,O
B:A,C,E,K
C:F,A,D,I
D:A,E,F,L
E:B,C,D,M,L
F:A,B,C,D,E,O,M
G:A,C,D,E,F
H:A,C,D,E,O
I:A,O
J:B,O
K:A,C,D
L:D,E,F
M:E,F,G
O:A,H,I,J

求出哪些人两两之间有共同好友，及他俩的共同好友都有谁？

**思路**

Job1：
例：数据源 A:B,C,D,F,E,O
1. Map读取数据，输出<B:A>,<C:A>,<D:A>.....
2. Reduce合并数据得到<B:A,C,D> 说明 ACD的共同好友都有B

Job2:
数据源 <B:A,C,D>
1. Map读取数据并输出 <A,C:B>,<A,D:B>,<C,D:B> 两两输出共同好友 注意排序、去重
2. Reduce合并结果即可 输出 <A,C:B,D,F>....

**代码实现**


```java
package hadoop.mr.friends;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.Arrays;

/**
 * @author NikoBelic
 * @create 2017/5/23 12:58
 */
public class CommonFriends
{
    public static void main(String[] args) throws IOException, URISyntaxException, InterruptedException, ClassNotFoundException
    {
        String inputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/friends/result";
        String outputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/friends/result2";
        Configuration conf = new Configuration();
        //conf.set("mapreduce.framework.name", "yarn");
        //conf.set("yarn.resourcemanager.hostname", "hadoop1");

        Job job = Job.getInstance(conf);
        //job.setUser("NikoBelic");
        //job.setJarByClass(FlowCount.class);

        job.setMapperClass(FriendsMapperStepTwo.class);
        job.setReducerClass(FriendsReducerStepOne.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        FileInputFormat.setInputPaths(job, new Path(inputPath));

        FileSystem fs = FileSystem.get(new URI(outputPath), conf, "NikoBelic");
        if (fs.exists(new Path(outputPath)))
        {
            fs.delete(new Path(outputPath));
        }
        FileOutputFormat.setOutputPath(job, new Path(outputPath));

        job.waitForCompletion(true);
    }
}


class FriendsMapperStepOne extends Mapper<LongWritable, Text, Text, Text>
{
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String host = value.toString().split(":")[0];
        String[] friends = value.toString().split(":")[1].split(",");
        for (String friend : friends)
        {
            context.write(new Text(friend), new Text(host));
        }
    }
}


class FriendsReducerStepOne extends Reducer<Text, Text, Text, Text>
{
    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException
    {
        StringBuilder sb = new StringBuilder();
        for (Text value : values)
        {
            sb.append(value).append(",");
        }
        context.write(key, new Text(sb.deleteCharAt(sb.length() - 1).toString()));
    }
}

class FriendsMapperStepTwo extends Mapper<LongWritable, Text, Text, Text>
{
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String valueStr = value.toString();
        String commonFriend = valueStr.split("\t")[0];
        String[] hosts = valueStr.split("\t")[1].split(",");
        for (int i = 0; i < hosts.length - 1; i++)
        {
            for (int j = i + 1; j < hosts.length; j++)
            {
                context.write(new Text(sortRelation(hosts[i] + "," + hosts[j])),new Text(commonFriend));
            }
        }
    }

    private String sortRelation(String relation)
    {
        String[] friends = relation.split(",");
        Arrays.sort(friends);
        return (friends[0] + "," + friends[1]).intern();
    }
}

//
//class FriendsReducerStepTwo extends Reducer<Text, Text, Text, Text>
//{
//    @Override
//    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException
//    {
//        StringBuilder sb = new StringBuilder();
//        for (Text value : values)
//        {
//            sb.append(value).append(",");
//        }
//        context.write(key, new Text(sb.deleteCharAt(sb.length() - 1).toString()));
//    }
//}
//

```

**Job1输出结果**

```
A	I,K,C,B,G,F,H,O,D
B	A,F,J,E
C	A,E,B,H,F,G,K
D	G,C,K,A,L,F,E,H
E	G,M,L,H,A,F,B,D
F	L,M,D,C,G,A
G	M
H	O
I	O,C
J	O
K	B
L	D,E
M	E,F
O	A,H,I,J,F

```

**Job2输出结果**


```
A,B	C,E
A,C	D,F
A,D	F,E
A,E	B,D,C
A,F	C,B,O,D,E
A,G	F,D,E,C
A,H	D,O,E,C
A,I	O
A,J	O,B
A,K	D,C
A,L	E,D,F
A,M	F,E
B,C	A
B,D	E,A
B,E	C
B,F	E,A,C
B,G	C,A,E
B,H	E,C,A
B,I	A
B,K	C,A
B,L	E
B,M	E
B,O	A
C,D	F,A
C,E	D
C,F	D,A
C,G	D,F,A
C,H	A,D
C,I	A
C,K	D,A
C,L	D,F
C,M	F
C,O	I,A
D,E	L
D,F	A,E
D,G	E,A,F
D,H	E,A
D,I	A
D,K	A
D,L	E,F
D,M	E,F
D,O	A
E,F	B,M,D,C
E,G	D,C
E,H	D,C
E,J	B
E,K	D,C
E,L	D
F,G	C,D,A,E
F,H	A,E,O,C,D
F,I	O,A
F,J	O,B
F,K	A,C,D
F,L	E,D
F,M	E
F,O	A
G,H	A,D,E,C
G,I	A
G,K	C,A,D
G,L	F,D,E
G,M	F,E
G,O	A
H,I	A,O
H,J	O
H,K	A,D,C
H,L	D,E
H,M	E
H,O	A
I,J	O
I,K	A
I,O	A
K,L	D
K,O	A
L,M	F,E

```

# 5 倒排索引建立

**需求：**
有大量的文本（文档、网页），需要建立搜索索引

假设有三个文件
a.txt  

```
hello tom
hello jerry
hello tom

```

b.txt

```
hello jerry
hello jerry
tom jerry
```

c.txt

```
hello jerry
hello tom
```

要求得到key与文件名之间的索引，并按词频倒排序

**思路**

Job1：

1. Map读取文件，输出<关键词:文件名> => <hello--a.txt,1>,<hello--a.txt,1>,<hello--a.txt,1><hello--b.txt,1>...
2. Reduce合并文件 输出<hello--a.txt:3>,<hello-b.txt:2>,<hello-c.txt:2>>....

Job2:

1. Map读取文件，切割、合并文件字符串  输出 <hello:a.txt,3> <hello:b.txt,2> <hello:c.txt,1>
2. Reduce合并关键词并按词频倒排序 输出 <hello:a.txt,3  b.txt,2 c.txt,1>

**代码实现**


```java
package hadoop.mr.reverseindex;

import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.jobcontrol.ControlledJob;
import org.apache.hadoop.mapreduce.lib.jobcontrol.JobControl;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
import java.net.URI;

/**
 * @author NikoBelic
 * @create 2017/5/24 15:07
 */
public class ReverseIndex
{
    public static void main(String[] args) throws Exception
    {
        String inputPath1 = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/revers_index";
        String outputPath1 = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/revers_index/result1";
        String inputPath2 = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/revers_index/result1/part-r-00000";
        String outputPath2 = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/revers_index/result2";
        org.apache.hadoop.conf.Configuration conf = new org.apache.hadoop.conf.Configuration();
        //conf.set("mapreduce.framework.name", "yarn");
        //conf.set("yarn.resourcemanager.hostname", "hadoop1");

        Job job1 = Job.getInstance(conf,"Job1");


        job1.setMapperClass(ReverseIndexMapperStepOne.class);
        job1.setReducerClass(ReverseIndexReducerStepOne.class);
        job1.setMapOutputKeyClass(Text.class);
        job1.setMapOutputValueClass(LongWritable.class);

        job1.setOutputKeyClass(Text.class);
        job1.setOutputValueClass(LongWritable.class);

        FileInputFormat.setInputPaths(job1, new Path(inputPath1));
        FileSystem fs = FileSystem.get(new URI(outputPath1), conf, "NikoBelic");
        if (fs.exists(new Path(outputPath1)))
        {
            fs.delete(new Path(outputPath1));
        }
        FileOutputFormat.setOutputPath(job1, new Path(outputPath1));

        //job1.waitForCompletion(true);


        Job job2 = Job.getInstance(conf,"Job2");
        job2.setMapperClass(ReverseIndexMapperStepTwo.class);
        job2.setReducerClass(ReverseIndexReducerStepTwo.class);

        job2.setMapOutputKeyClass(Text.class);
        job2.setMapOutputValueClass(Text.class);

        job2.setOutputKeyClass(Text.class);
        job2.setOutputValueClass(Text.class);


        FileInputFormat.setInputPaths(job2, new Path(inputPath2));
        if (fs.exists(new Path(outputPath2)))
        {
            fs.delete(new Path(outputPath2));
        }
        FileOutputFormat.setOutputPath(job2, new Path(outputPath2));

        job1.waitForCompletion(true);

        ControlledJob ctrlJob1 = new ControlledJob(conf);
        ctrlJob1.setJob(job1);

        ControlledJob ctrlJob2 = new ControlledJob(conf);
        ctrlJob2.setJob(job2);

        JobControl jobControl = new JobControl("MyControl");
        jobControl.addJob(ctrlJob1);
        jobControl.addJob(ctrlJob2);
        jobControl.run();
        System.out.println(jobControl.getFailedJobList());
        jobControl.stop();
    }
    static class ReverseIndexMapperStepOne extends Mapper<LongWritable, Text, Text, LongWritable>
    {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
        {
            FileSplit fileSplit = (FileSplit) context.getInputSplit();
            String[] fields = value.toString().split(" ");
            String fileName = fileSplit.getPath().getName();
            for (String field : fields)
            {
                context.write(new Text(field + "--" + fileName), new LongWritable(1L));
            }

        }
    }

    static class ReverseIndexReducerStepOne extends Reducer<Text, LongWritable, Text, LongWritable>
    {
        @Override
        protected void reduce(Text key, Iterable<LongWritable> values, Context context) throws IOException, InterruptedException
        {
            Long totalCount = 0L;
            for (LongWritable value : values)
            {
                totalCount += value.get();
            }
            context.write(key, new LongWritable(totalCount));
        }
    }

    static class ReverseIndexMapperStepTwo extends Mapper<LongWritable, Text, Text, Text>
    {
        @Override
        protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
        {
            String[] fields = value.toString().split("--");
            String word = fields[0];
            String freq = fields[1];

            context.write(new Text(word),new Text(freq));

        }
    }

    static class ReverseIndexReducerStepTwo extends Reducer<Text, Text, Text, Text>
    {
        @Override
        protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException
        {
            StringBuilder sb = new StringBuilder();
            for (Text value : values)
            {
                sb.append(value).append("   ");
            }
            context.write(key, new Text(sb.toString()));
        }
    }

}

```

**Job1输出**


```
hello--a.txt	3
hello--b.txt	2
hello--c.txt	2
jerry--a.txt	1
jerry--b.txt	3
jerry--c.txt	1
tom--a.txt	2
tom--b.txt	1
tom--c.txt	1

```


**Job2输出**

```
hello	c.txt	2   b.txt	2   a.txt	3   
jerry	c.txt	1   b.txt	3   a.txt	1   
tom	c.txt	1   b.txt	1   a.txt	2   

```

# 6 最大订单金额

有如下订单数据(订单号、商品id、订单金额)
```
Order_0000001,Pdt_01,222.8
Order_0000001,Pdt_05,25.8
Order_0000002,Pdt_05,325.8
Order_0000002,Pdt_03,522.8
Order_0000002,Pdt_04,122.4
Order_0000003,Pdt_01,222.8
Order_0000003,Pdt_01,322.8
```
现在需要求出每一个订单中成交金额最大的一笔交易

**传统方法:**
Map输出<OrderId,price>  Reduce只输出最大金额的订单信息

**新方法:**
自定义GroupingComparator

1、利用“订单id和成交金额”作为key，可以将map阶段读取到的所有订单数据按照id分区，按照金额排序，发送到reduce
2、在reduce端利用groupingcomparator将订单id相同的kv聚合成组，然后取第一个即是最大值

**代码实现**


```java
package hadoop.mr.orders;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.net.URI;

/**
 * @author NikoBelic
 * @create 2017/6/4 22:18
 */
public class Orders
{
    public static void main(String[] args) throws Exception
    {
        String inputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/orders";
        String outputPath = "/Users/lixiwei-mac/Documents/IdeaProjects/bigdatalearning/src/main/java/hadoop/data/orders/result";
        Configuration conf = new Configuration();

        Job job = Job.getInstance(conf);
        //job.setUser("NikoBelic");
        //job.setJarByClass(FlowCount.class);

        job.setMapperClass(OrdersMapper.class);
        job.setReducerClass(OrdersReducer.class);
        job.setPartitionerClass(OrdersPartitioner.class);
        job.setNumReduceTasks(3);

        job.setMapOutputKeyClass(OrderDO.class);
        job.setMapOutputValueClass(NullWritable.class);

        job.setOutputKeyClass(OrderDO.class);
        job.setOutputValueClass(NullWritable.class);

        job.setGroupingComparatorClass(OrderIdGroupingComparator.class);

        FileInputFormat.setInputPaths(job, new Path(inputPath));

        FileSystem fs = FileSystem.get(new URI(outputPath), conf, "NikoBelic");
        if (fs.exists(new Path(outputPath)))
        {
            fs.delete(new Path(outputPath));
        }
        FileOutputFormat.setOutputPath(job, new Path(outputPath));

        job.waitForCompletion(true);
    }
}


class OrdersMapper extends Mapper<LongWritable, Text, OrderDO, NullWritable>
{
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
    {
        String[] fields = value.toString().split(",");
        context.write(new OrderDO(fields[0], fields[1], Double.valueOf(fields[2])), NullWritable.get());

    }
}

class OrdersPartitioner extends Partitioner<OrderDO, NullWritable>
{
    @Override
    public int getPartition(OrderDO orderDO, NullWritable nullWritable, int i)
    {
        System.out.println((orderDO.getOrderNo().hashCode() & Integer.MAX_VALUE) % i);
        return (orderDO.getOrderNo().hashCode() & Integer.MAX_VALUE) % i;
    }
}

class OrdersReducer extends Reducer<OrderDO, NullWritable, OrderDO, NullWritable>
{
    @Override
    protected void reduce(OrderDO key, Iterable<NullWritable> values, Context context) throws IOException, InterruptedException
    {
        context.write(key, NullWritable.get());
    }
}

class OrderIdGroupingComparator extends WritableComparator
{
    public OrderIdGroupingComparator()
    {
        super(OrderDO.class, true);
    }

    @Override
    public int compare(WritableComparable a, WritableComparable b)
    {
        OrderDO o1 = (OrderDO) a;
        OrderDO o2 = (OrderDO) b;
        return o1.getOrderNo().compareTo(o2.getOrderNo());

    }
}

class OrderDO implements WritableComparable<OrderDO>
{
    private String orderNo;
    private String productName;
    private Double price;

    public OrderDO()
    {
    }

    public OrderDO(String orderNo, String productName, Double price)
    {
        this.orderNo = orderNo;
        this.productName = productName;
        this.price = price;
    }

    public String getOrderNo()
    {
        return orderNo;
    }

    public void setOrderNo(String orderNo)
    {
        this.orderNo = orderNo;
    }

    public String getProductName()
    {
        return productName;
    }

    public void setProductName(String productName)
    {
        this.productName = productName;
    }

    public Double getPrice()
    {
        return price;
    }

    public void setPrice(Double price)
    {
        this.price = price;
    }

    @Override
    public int compareTo(OrderDO o)
    {
        if (this.orderNo.compareTo(o.getOrderNo()) == 0)
        {
            return -this.price.compareTo(o.getPrice());
        } else
        {
            return this.orderNo.compareTo(o.getOrderNo());
        }
    }

    @Override
    public void write(DataOutput dataOutput) throws IOException
    {
        dataOutput.writeUTF(this.orderNo);
        dataOutput.writeUTF(this.productName);
        dataOutput.writeDouble(this.price);
    }

    @Override
    public void readFields(DataInput dataInput) throws IOException
    {
        this.orderNo = dataInput.readUTF();
        this.productName = dataInput.readUTF();
        this.price = dataInput.readDouble();
    }

    @Override
    public String toString()
    {
        return "OrderDO{" + "orderNo='" + orderNo + '\'' + ", productName='" + productName + '\'' + ", price=" + price + '}';
    }
}
```

**输出**

三个文件

```
OrderDO{orderNo='Order_0000001', productName='Pdt_01', price=222.8}
```

```
OrderDO{orderNo='Order_0000002', productName='Pdt_03', price=522.8}
```

```
OrderDO{orderNo='Order_0000003', productName='Pdt_01', price=322.8}
```

