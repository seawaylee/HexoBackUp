---
title: Hive学习笔记（一）- 详解Hive
date: 2017-07-02 23:35:33
tags: [大数据,Hive]
---


# 1 Hive基础知识

## 1.1 简介
**什么是Hive**
Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供类SQL查询功能。

**为什么使用Hive**
    
- 直接使用Hadoop所面临的问题
    - 学习成本高
    - 一般项目周期要求短
    - MapReduce实现复杂的查询呢逻辑开发难度较大

- 为什么要使用Hive
    - 操作接口采用类SQL语法，可以快速开发
    - 避免编写MR，减少学习成本
    - 扩展功能很方便

**Hive的特点**

- 可扩展： Hive可以自由的扩展集群规模，一般情况下不需要重启服务
- 延展性： Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数
- 容错： 良好的容错性，节点出现问题SQL仍然可以完成执行

<!--more-->

## 1.2 架构

![](14983216998950.jpg)

JobTracker是Hadoop1.x中的ResouceManager + AppMaster
TaskTracker相当于NodeManager + YarnChild

### 1.2.1 基本组成

- 用户接口：CLI、JDBC/ODBC、WebGUI
    - CLI是SHELL命令行
    - JDBC、ODBC是Hive的**Java**实现，与传统的JDBC类似
    - WebGUI是通过**浏览器访问**Hive
- 元数据存储：Mysql、Derby等
    - Hive将**元数据**存储在数据库中。Hive中的元数据包括**表名、列、分区及其属性、表的属性（是否为外部表等）、表数据所在目录**等。
- 解释器、编译器、优化器、执行器
    - 完成HQL查询语句 **词法分析、语法分析、编译、优化以及查询计划**的生成。生成的查询计划存储在HDFS中，并在随后由MapReduce调用执行。


## 1.3 Hive与Hadoop的关系

Hive利用HDFS存储数据，利用MapReduce查询数据

![](14986658358207.jpg)

## 1.4 Hive与传统数据库对比

Hive具有SQL数据库的外表，但应用应用场景完全不同，Hive只适合用来做批量的数据统计分析

|  | Hive | RDBMS |
| --- | --- | --- |
| 查询语言 | HQL | SQL |
| 数据存储 | HDFS | LOCAL FS |
| 执行 | MapReduce | Executor |
| 执行延迟 | 高 | 低 |
| 处理数据规模 | 大 | 小 |
| 索引 | 位图索引 | 复杂索引 |

## 1.5 Hive的数据存储

1. Hive中所有的数据都**存储在HDFS**中，没有专门的数据存储格式（可支持Text，SequenceFile，ParquetFile，RCFILE等）
2. 只需要在**创建表的时候**告诉Hive数据中的**列分隔符**和**行分隔符**，Hive就可以解析数据。
3. Hive中包含以下数据模型：DB、Table、External Table、Partition、Bucket
     - DB：在HDFS中表现为 **${hive.metastore.warehouse.dir} 目录下**的文件夹
     - Table：在HDFS中表现所属**DB目录下**的文件夹
     - External Table：外部表，与Table类似，不过其数据存放的**位置**可以在**任意指定路径**。
         - Table表：删除表后，HDFS上的**文件都删除了**
         - External外部表：删除表后，HDFS上的**文件没有删除**，只是把表删除了
    - Partition：在HDFS中表现为**Table目录下的子目录**
    - Bucket：桶，在HDFS中表现为同一个表目录下根据**Hash散列**之后的多个文件，会根据不同的文件把数据放到**不同的文件夹中**



## 1.6 Hive的安装的部署

### 1.6.1 准备工作

1. 安装mysql
2. 下载、解压Hive
3. 修改hive-site.xml，配置mysql


```xml
<configuration>
<property>
<name>javax.jdo.option.ConnectionURL</name>
<value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
<description>JDBC connect string for a JDBC metastore</description>
</property>

<property>
<name>javax.jdo.option.ConnectionDriverName</name>
<value>com.mysql.jdbc.Driver</value>
<description>Driver class name for a JDBC metastore</description>
</property>

<property>
<name>javax.jdo.option.ConnectionUserName</name>
<value>root</value>
<description>username to use against metastore database</description>
</property>

<property>
<name>javax.jdo.option.ConnectionPassword</name>
<value>MYSQL的密码</value>
<description>password to use against metastore database</description>
</property>
</configuration>


```


### 1.6.2 使用Hive

- Hive的Shell交互模式
    - `bin/hive` 启动hive （与mysql操作方式一样）
- Hive Thrift服务
    - 在安装有hive的主机上后台启动 `nohup ./hiveserver2 1> hiverserver.log 2>&1 &`
    - 在任意子hadoop子节点上用beeline连接hiveserver 
        - 方式1
            - `./beeline`
            - `!connect jdbc:hive2://hadoop1:10000 NikoBelic`
        - 方式2
            -  `./beeline -u jdbc:hive2://hadoop1:10000 -n NikoBelic`
- 使用sql语句疯狂输出吧

# 2 Hive基本操作

## 2.1 DDL操作

### 2.1.1 创建表

**建表语法**

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
   [(col_name data_type [COMMENT col_comment], ...)] 
   [COMMENT table_comment] 
   [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
   [CLUSTERED BY (col_name, col_name, ...) 
   [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
   [ROW FORMAT row_format] 
   [STORED AS file_format] 
   [LOCATION hdfs_path]
```

**说明：**
1. CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXISTS 选项来忽略这个异常。

2. EXTERNAL关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION），Hive 创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。

3. PARTITIONED 表示根据某一个key(不在create table里面)对数据进行分区，体现在HDFS上就是 table目录下有n个不同的分区文件夹(country=China,country=USA)

4. ROW FORMAT 
    DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char] 
   [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char] 
   | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
用户在建表的时候可以自定义 SerDe 或者使用自带的 SerDe。如果没有指定 ROW FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的 SerDe，Hive通过 SerDe 确定表的具体的列的数据。

5. STORED AS 
SEQUENCEFILE|TEXTFILE|RCFILE
如果文件数据是纯文本，可以使用 STORED AS TEXTFILE。如果数据需要压缩，使用 STORED AS SEQUENCEFILE。

6. CLUSTERED BY
对于每一个表（table）或者分区， Hive可以进一步组织成桶，也就是说桶是更为细粒度的数据范围划分。Hive也是 针对某一列进行桶的组织。Hive采用对列值哈希，然后除以桶的个数求余的方式决定该条记录存放在哪个桶当中。 
把表（或者分区）组织成桶（Bucket）有两个理由：
- （1）获得更高的查询处理效率。桶为表加上了额外的结构，Hive 在处理有些查询时能利用这个结构。具体而言，连接两个在（包含连接列的）相同列上划分了桶的表，可以使用 Map 端连接 （Map-side join）高效的实现。比如JOIN操作。对于JOIN操作两个表有一个相同的列，如果对这两个表都进行了桶操作。那么将保存相同列值的桶进行JOIN操作就可以，可以大大较少JOIN的数据量。
- （2）使取样（sampling）更高效。在处理大规模数据集时，在开发和修改查询的阶段，如果能在数据集的一小部分数据上试运行查询，会带来很多方便。



**实例**


```sql
show databases;
use hive_test_db;

create table stu_buck(Sno int,Sname string,Sex string,Sage int,Sdept string)
clustered by(Sno) 
sorted by(Sno DESC)
into 4 buckets
row format delimited
fields terminated by ',';
```


设置变量,设置分桶为true, 设置reduce数量是分桶的数量个数
set hive.enforce.bucketing = true;
set mapreduce.job.reduces=4;



### 2.1.2 修改表

- 增加/删除分区


```sql
ALTER TABLE table_name ADD [IF NOT EXISTS] partition_spec [ LOCATION 'location1' ] partition_spec [ LOCATION 'location2' ] ...
partition_spec:
: PARTITION (partition_col = partition_col_value, partition_col = partiton_col_value, ...)

ALTER TABLE table_name DROP partition_spec, partition_spec,...
```

**实例**


```sql
alter table student_p add partition(part='a') partition(part='b');
```


- 重命名表


```sql
ALTER TABLE table_name RENAME TO new_table_name
```

- 增加/更新列


```sql
ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
```
ADD是代表新增一字段，字段位置在所有列后面(partition列前)，REPLACE则是表示替换表中所有字段。


```sql
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
```

### 2.1.3 显示命令


```sql
show tables
show databases
show partitions
show functions
desc extended t_name;
desc formatted table_name;
```

## 2.2 DML操作

### 2.2.1 Load


```
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO 
TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
```


```sql
load data local inpath '/Users/lixiwei-mac/app/data/hive_tmp/metrics.data' into table metrics;
```

1. Load 操作只是单纯的复制/移动操作，将数据文件移动到 Hive 表对应的位置。

2. filepath：
    相对路径，例如：project/data1 
    绝对路径，例如：/user/hive/project/data1 
    包含模式的完整 URI，列如：
    hdfs://namenode:9000/user/hive/project/data1

3. LOCAL关键字
    如果指定了 LOCAL， load 命令会去查找本地文件系统中的 filepath。
    如果没有指定 LOCAL 关键字，则根据inpath中的uri查找文件


4. OVERWRITE 关键字
    如果使用了 OVERWRITE 关键字，则目标表（或者分区）中的内容会被删除，然后再将 filepath 指向的文件/目录中的内容添加到表/分区中。 
    如果目标表（分区）已经有一个文件，并且文件名和 filepath 中的文件名冲突，那么现有的文件会被新文件所替代。 
    
### 2.2.2 Insert


```sql
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement
```

**Multiple inserts:**

```sql
FROM from_statement 
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 
[INSERT OVERWRITE TABLE tablename2 [PARTITION ...] select_statement2] ...
```
**Dynamic partition inserts:**


```sql
INSERT OVERWRITE TABLE tablename PARTITION (partcol1[=val1], partcol2[=val2] ...) select_statement FROM from_statement
```


```sql
insert into table metrics_buck select * from metrics distribute by (type);
```

导出表数据


```sql
INSERT OVERWRITE [LOCAL] DIRECTORY directory1 SELECT ... FROM ...
```


### 2.2.3 Select


```sql
SELECT [ALL | DISTINCT] select_expr, select_expr, ... 
FROM table_reference
[WHERE where_condition] 
[GROUP BY col_list [HAVING condition]] 
[CLUSTER BY col_list 
  | [DISTRIBUTE BY col_list] [SORT BY| ORDER BY col_list] 
] 
[LIMIT number]
```

注：

1. order by 会对输入做全局排序，因此只有一个reducer，会导致当输入规模较大时，需要较长的计算时间。
2. sort by不是全局排序，其在数据进入reducer前完成排序。因此，如果用sort by进行排序，并且设置mapred.reduce.tasks>1，则sort by只保证每个reducer的输出有序，不保证全局有序。
3. distribute by根据distribute by指定的内容将数据分到同一个reducer。
4. Cluster by 除了具有Distribute by的功能外，还会对该字段进行排序。因此，常常认为cluster by = distribute by + sort by


## 2.3 Hive Join


```sql
join_table:
  table_reference JOIN table_factor [join_condition]
  | table_reference {LEFT|RIGHT|FULL} [OUTER] JOIN table_reference join_condition
  | table_reference LEFT SEMI JOIN table_reference join_condition

```


Hive 支持等值连接（equality joins）、外连接（outer joins）和（left/right joins）。Hive **不支持非等值的连接**，因为非等值连接非常难转化到 map/reduce 任务。另外，Hive 支持多于 2 个表的连接。

**写 join 查询时，需要注意几个关键点：**

**1、只支持等值join**
例如： 

```sql
 SELECT a.* FROM a JOIN b ON (a.id = b.id)
  SELECT a.* FROM a JOIN b
    ON (a.id = b.id AND a.department = b.department)
```

是正确的，然而:

```sql
SELECT a.* FROM a JOIN b ON (a.id>b.id)
```

是错误的。

**2、可以 join 多于 2 个表。**
例如
  
```
SELECT a.val, b.val, c.val FROM a JOIN b
    ON (a.key = b.key1) JOIN c ON (c.key = b.key2)
```

如果join中多个表的 join key 是同一个，则 join 会被转化为单个 map/reduce 任务，例如：
  
```sql
SELECT a.val, b.val, c.val FROM a JOIN b
    ON (a.key = b.key1) JOIN c
    ON (c.key = b.key1)
```
被转化为单个 map/reduce 任务，因为 join 中只使用了 b.key1 作为 join key。

```sql
SELECT a.val, b.val, c.val FROM a JOIN b ON (a.key = b.key1)
  JOIN c ON (c.key = b.key2)
```
而这一 join 被转化为 2 个 map/reduce 任务。因为 b.key1 用于第一次 join 条件，而 b.key2 用于第二次 join。
   
**3、join 时，每次 map/reduce 任务的逻辑：**
    
reducer 会**缓存 join 序列中除了最后一个表的所有表的记录**，再通过最后一个表将结果序列化到文件系统。这一实现有助于在 reduce 端减少内存的使用量。实践中，应该**把最大的那个表写在最后**（否则会因为缓存浪费大量内存）。例如：

```sql
SELECT a.val, b.val, c.val FROM a
    JOIN b ON (a.key = b.key1) JOIN c ON (c.key = b.key1)
```

所有表都使用同一个 join key（使用 1 次 map/reduce 任务计算）。Reduce 端会缓存 a 表和 b 表的记录，然后每次取得一个 c 表的记录就计算一次 join 结果，类似的还有：

```sql  
SELECT a.val, b.val, c.val FROM a
    JOIN b ON (a.key = b.key1) JOIN c ON (c.key = b.key2)
```

这里用了 2 次 map/reduce 任务。第一次缓存 a 表，用 b 表序列化；第二次缓存第一次 map/reduce 任务的结果，然后用 c 表序列化。

**4、LEFT，RIGHT 和 FULL OUTER 关键字用于处理 join 中空记录的情况**
例如：
  
```sql
SELECT a.val, b.val FROM 
a LEFT OUTER  JOIN b ON (a.key=b.key)
```

- 对应所有 a 表中的记录都有一条记录输出。输出的结果应该是 a.val, b.val，当 a.key=b.key 时，而当 b.key 中找不到等值的 a.key 记录时也会输出:
    a.val, NULL
    所以 a 表中的所有记录都被保留了；
    “a RIGHT OUTER JOIN b”会保留所有 b 表的记录。

- Join 发生在 WHERE 子句之前。如果你想限制 join 的输出，应该在 WHERE 子句中写过滤条件——或是在 join 子句中写。这里面一个容易混淆的问题是表分区的情况：
  
```sql
SELECT a.val, b.val FROM a
  LEFT OUTER JOIN b ON (a.key=b.key)
  WHERE a.ds='2009-07-07' AND b.ds='2009-07-07'
```
会 join a 表到 b 表（OUTER JOIN），列出 a.val 和 b.val 的记录。WHERE 从句中可以使用其他列作为过滤条件。但是，如前所述，如果 b 表中找不到对应 a 表的记录，b 表的所有列都会列出 NULL，包括 ds 列。也就是说，join 会过滤 b 表中不能找到匹配 a 表 join key 的所有记录。这样的话，LEFT OUTER 就使得查询结果与 WHERE 子句无关了。解决的办法是在 OUTER JOIN 时使用以下语法：

```sql  
SELECT a.val, b.val FROM a LEFT OUTER JOIN b
  ON (a.key=b.key AND
      b.ds='2009-07-07' AND
      a.ds='2009-07-07')

```

这一查询的结果是预先在 join 阶段过滤过的，所以不会存在上述问题。这一逻辑也可以应用于 RIGHT 和 FULL 类型的 join 中。

- Join 是不能交换位置的。无论是 LEFT 还是 RIGHT join，都是左连接的。
  
```sql
SELECT a.val1, a.val2, b.val, c.val
  FROM a
  JOIN b ON (a.key = b.key)
  LEFT OUTER JOIN c ON (a.key = c.key)
```
先 join a 表到 b 表，丢弃掉所有 join key 中不匹配的记录，然后用这一中间结果和 c 表做 join。这一表述有一个不太明显的问题，就是当一个 key 在 a 表和 c 表都存在，但是 b 表中不存在的时候：整个记录在第一次 join，即 a JOIN b 的时候都被丢掉了（包括a.val1，a.val2和a.key），然后我们再和 c 表 join 的时候，如果 c.key 与 a.key 或 b.key 相等，就会得到这样的结果：NULL, NULL, NULL, c.val


## 2.4 HQL小结


```sql
show databases;
show tables;
desc test;
```

-------------
### 2.4.1 分桶表示例

- 创建分桶表

    ```sql
    
    drop table stu_buck;
    create table stu_buck(Sno int,Sname string,Sex string,Sage int,Sdept string)
    clustered by(Sno) 
    sorted by(Sno DESC)
    into 4 buckets
    row format delimited
    fields terminated by ',';
    ```

- 设置变量,设置分桶为true, 设置reduce数量是分桶的数量个数

    - set hive.enforce.bucketing = true;
    - set mapreduce.job.reduces=4;
    - insert overwrite table student_buck
    - select * from student cluster by(Sno) sort by(Sage);  报错,cluster 和 sort 不能共存

- 往创建的分通表插入数据(插入数据需要是已分桶, 且排序的)
    - 可以使用distribute by(sno) sort by(sno asc)   或是排序和分桶的字段相同的时候使用Cluster by(字段)
    - 注意使用cluster by  就等同于分桶+排序(sort)

    ```    sql
    insert into table stu_buck
    select Sno,Sname,Sex,Sage,Sdept from student distribute by(Sno) sort by(Sno asc);
    
    insert overwrite table stu_buck
    select * from student distribute by(Sno) sort by(Sno asc);
    
    insert overwrite table stu_buck
    select * from student cluster by(Sno);
    
    ```
------------------------

### 2.4.2 保存select查询结果的几种方式：

1. 将查询结果保存到一张新的hive表中

   ```sql
   create table t_tmp
   as
   select * from t_p;
   
   ```
    
2. 将查询结果保存到一张已经存在的hive表中

   ```sql
   insert into  table t_tmp
   select * from t_p;
   
   ```
3. 将查询结果保存到指定的文件目录（可以是本地，也可以是hdfs）

   ```sql
   insert overwrite local directory '/home/hadoop/test'
   select * from t_p;
   
   insert overwrite directory '/aaa/test'
   select * from t_p;

   ```


-----------------------------------

### 2.4.3 关于hive中的各种join

1. 准备数据

    1,a
    2,b
    3,c
    4,d
    7,y
    8,u
    
    2,bb
    3,cc
    7,yy
    9,pp



2. 建表：


    ```sql
    create table a(id int,name string)
    row format delimited fields terminated by ',';
    
    create table b(id int,name string)
    row format delimited fields terminated by ',';
    
    ```
3. 导入数据

    ```sql
        load data local inpath '/home/hadoop/a.txt' into table a;
        load data local inpath '/home/hadoop/b.txt' into table b;  
    ```

4. 实验

    - inner join


        ```sql
        select * from a inner join b on a.id=b.id;
        +-------+---------+-------+---------+--+
        | a.id  | a.name  | b.id  | b.name  |
        +-------+---------+-------+---------+--+
        | 2     | b       | 2     | bb      |
        | 3     | c       | 3     | cc      |
        | 7     | y       | 7     | yy      |
        +-------+---------+-------+---------+--+
    ```


   - left join
        
        ```sql
        select * from a left join b on a.id=b.id;
                +-------+---------+-------+---------+--+
                | a.id  | a.name  | b.id  | b.name  |
                +-------+---------+-------+---------+--+
                | 1     | a       | NULL  | NULL    |
                | 2     | b       | 2     | bb      |
                | 3     | c       | 3     | cc      |
                | 4     | d       | NULL  | NULL    |
                | 7     | y       | 7     | yy      |
                | 8     | u       | NULL  | NULL    |
                +-------+---------+-------+---------+--+
        
        ```

   - right join

        `select * from a right join b on a.id=b.id;`

   - outer join

        ```sql
        select * from a full outer join b on a.id=b.id;
        +-------+---------+-------+---------+--+
        | a.id  | a.name  | b.id  | b.name  |
        +-------+---------+-------+---------+--+
        | 1     | a       | NULL  | NULL    |
        | 2     | b       | 2     | bb      |
        | 3     | c       | 3     | cc      |
        | 4     | d       | NULL  | NULL    |
        | 7     | y       | 7     | yy      |
        | 8     | u       | NULL  | NULL    |
        | NULL  | NULL    | 9     | pp      |
        +-------+---------+-------+---------+--+
         
        ```
   - left semi join

        ```sql
        select * from a left semi join b on a.id = b.id;
        +-------+---------+--+
        | a.id  | a.name  |
        +-------+---------+--+
        | 2     | b       |
        | 3     | c       |
        | 7     | y       |
        +-------+---------+--+
        
        ```
        
-------------

### 2.4.4 其他

- 多重插入：


    ```sql
    from student
    insert into table student_p partition(part='a')
    select * where Sno<95011;
    insert into table student_p partition(part='a')
    select * where Sno<95011;
    
    ```


- 导出数据到本地

    ```sql
    insert overwrite local directory '/home/hadoop/student.txt'
    select * from student;
    
    ```

- UDF案例


    ```sql
    create table rat_json(line string) row format delimited;
    load data local inpath '/home/hadoop/rating.json' into table rat_json;
    
    drop table if exists t_rating;
    create table t_rating(movieid string,rate int,timestring string,uid string)
    row format delimited fields terminated by '\t';
    
    insert overwrite table t_rating
    select split(parsejson(line),'\t')[0]as movieid,split(parsejson(line),'\t')[1] as rate,split(parsejson(line),'\t')[2] as timestring,split(parsejson(line),'\t')[3] as uid from rat_json limit 10;
    
    ```


- 内置jason函数


    ```sql
    select get_json_object(line,'$.movie') as moive,get_json_object(line,'$.rate') as rate  from rat_json limit 10;
    
    ```


- transform案例

    1. 先加载rating.json文件到hive的一个原始表 rat_json

        ```sql
        create table rat_json(line string) row format delimited;
        load data local inpath '/home/hadoop/rating.json' into table rat_json;
        
        ```
    2. 需要解析json数据成四个字段，插入一张新的表 t_rating

        ```sql
        insert overwrite table t_rating
        select get_json_object(line,'$.movie') as moive,get_json_object(line,'$.rate') as rate  from rat_json;
        
        ```
3. 使用transform+python的方式去转换unixtime为weekday

    - 先编辑一个python脚本文件
        
        `vi weekday_mapper.py`

        ```python
        
        #!/bin/python
        import sys
        import datetime
        
        for line in sys.stdin:
          line = line.strip()
          movieid, rating, unixtime,userid = line.split('\t')
          weekday = datetime.datetime.fromtimestamp(float(unixtime)).isoweekday()
          print '\t'.join([movieid, rating, str(weekday),userid])
    ```

    - 保存文件
    - 然后，将文件加入hive的classpath：
        - hive>add FILE /home/hadoop/weekday_mapper.py;
        - hive>create TABLE u_data_new as

    ```sql
    SELECT
      TRANSFORM (movieid, rate, timestring,uid)
      USING 'python weekday_mapper.py'
      AS (movieid, rate, weekday,uid)
    FROM t_rating;
    
    select distinct(weekday) from u_data_new limit 10;    
    ```















# 3 Hive Shell参数

## 3.1 Hive命令行

**语法结构**

hive [-hiveconf x=y]* [<-i filename>]* [<-f filename>|<-e query-string>] [-S]

**说明：**

-i 从文件初始化HQL。
-e从命令行执行指定的HQL 
-f 执行HQL脚本 
-v 输出执行的HQL语句到控制台 
-p <port> connect to Hive Server on port number 
-hiveconf x=y Use this to set hive/hadoop configuration variables.


## 3.2 Hive参数配置方式 


### 3.2.1 Hive参数大全

https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties

开发Hive应用时，不可避免地需要设定Hive的参数。设定Hive的参数可以调优HQL代码的执行效率，或帮助定位问题。然而实践中经常遇到的一个问题是，为什么设定的参数没有起作用？这通常是错误的设定方式导致的。

**对于一般参数，有以下三种设定方式：**

- 配置文件 
- 命令行参数 
- 参数声明 

### 3.2.2 配置文件

Hive的配置文件包括

- 用户自定义配置文件：$HIVE_CONF_DIR/hive-site.xml 
- 默认配置文件：$HIVE_CONF_DIR/hive-default.xml 

用户自定义配置会覆盖默认配置。
另外，Hive也会读入Hadoop的配置，因为Hive是作为Hadoop的客户端启动的，Hive的配置会覆盖Hadoop的配置。
配置文件的设定对本机启动的所有Hive进程都有效。

### 3.2.3 命令行参数

启动Hive（客户端或Server方式）时，可以在命令行添加-hiveconf param=value来设定参数，例如：
bin/hive -hiveconf hive.root.logger=INFO,console
这一设定对本次启动的Session（对于Server方式启动，则是所有请求的Sessions）有效。

### 3.2.4 参数声明

可以在HQL中使用SET关键字设定参数，例如：

```
set mapred.reduce.tasks=100;
```
这一设定的作用域也是session级的。

上述三种设定方式的优先级依次递增。即参数声明覆盖命令行参数，命令行参数覆盖配置文件设定。注意某些系统级的参数，例如log4j相关的设定，必须用前两种方式设定，因为那些参数的读取在Session建立以前已经完成了。





# 4 Hive函数

**创建测试表**

- `vim dual.data` 只写一个空格
- 创建表 
    - `use school;`
    - `create table dual(id int);`
- `load data local inpath '/Users/lixiwei-mac/app/data/hive_tmp/dual.data' into table dual;` 导入数据
- 测试
    - `select substr('NikoBelic',0,4) from dual;`
        
        ```text
        +-------+--+
        |  _c0  |
        +-------+--+
        | Niko  |
        +-------+--+
        ```


## 4.1 内置运算符

[内容较多，见《Hive官方文档》](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-StringOperators)

## 4.2 内置函数

[内容较多，见《Hive官方文档》](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-StringOperators)

## 4.3 Hive自定义函数和Transform

当Hive提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数（UDF：user-defined function）。

### 4.3.1 自定义函数类别

- UDF  作用于单个数据行，产生一个数据行作为输出。（数学函数，字符串函数）
- UDAF（用户定义聚集函数）：接收多个输入数据行，并产生一个输出数据行。（count，max）

### 4.3.2 UDF开发实例

1. 先开发一个java类，继承UDF，并重载evaluate方法

    ```java
    package cn.itcast.bigdata.udf
    import org.apache.hadoop.hive.ql.exec.UDF;
    import org.apache.hadoop.io.Text;
    
    public final class Lower extends UDF{
    	public Text evaluate(final Text s){
    		if(s==null){return null;}
    		return new Text(s.toString().toLowerCase());
    	}
    }
    ```
    
2. 打成jar包上传到服务器
3. 将jar包添加到hive的classpath
    - hive>add JAR /home/hadoop/udf.jar;
    - 创建临时函数与开发好的java class关联
    - 即可在hql中使用自定义的函数strip 
    - Select strip(name),age from t_test;
4. 创建临时函数与开发好的java class关联
    - `Hive>create temporary function toprovince as 'cn.itcast.bigdata.udf.ToProvince';`
5. 即可在hql中使用自定义的函数strip 
    - `Select strip(name),age from t_test;`

### 4.3.3 Transform实现

* Hive的 TRANSFORM 关键字提供了在SQL中调用自写脚本的功能
* 适合实现Hive中没有的功能又不想写UDF的情况

```sql
CREATE TABLE u_data_new (
  movieid INT,
  rating INT,
  weekday INT,
  userid INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t';

add FILE weekday_mapper.py;

INSERT OVERWRITE TABLE u_data_new
SELECT
  TRANSFORM (movieid, rating, unixtime,userid)
  USING 'python weekday_mapper.py'
  AS (movieid, rating, weekday,userid)
FROM u_data;
```

* 使用示例1：下面这句sql就是借用了weekday_mapper.py对数据进行了处理.

```python
#!/bin/python
import sys
import datetime

for line in sys.stdin:
  line = line.strip()
  movieid, rating, unixtime,userid = line.split('\t')
  weekday = datetime.datetime.fromtimestamp(float(unixtime)).isoweekday()
  print '\t'.join([movieid, rating, str(weekday),userid])
```

* 其中weekday_mapper.py内容如下

```python
#!/bin/python
import sys
import datetime

for line in sys.stdin:
  line = line.strip()
  movieid, rating, unixtime,userid = line.split('\t')
  weekday = datetime.datetime.fromtimestamp(float(unixtime)).isoweekday()
  print '\t'.join([movieid, rating, str(weekday),userid])
```

* 使用示例2：下面的例子则是使用了shell的cat命令来处理数据


```sql
FROM invites a INSERT OVERWRITE TABLE events SELECT TRANSFORM(a.foo, a.bar) AS (oof, rab) USING '/bin/cat' WHERE a.ds > '2008-08-09';
```

# 5 遇到的问题问题

1. UDF 载入Jar成功 但添加function时ClassNotFound。

    - 如果确定create function xx as 'ClassPackageName' 中的ClassPackageName没有输入错误，那么可能是导出的Jar出现了问题。
    - Idea编辑器下，导出jar时不要选择 from moudles 而要选择empty,并创建META-INF，不要引入依赖jar包
    - 退出Hive服务重新连接并导入即可

2. 创建UDF方法出现异常  Unsupported major.minor version 52.0
    
    - Hadoop的JDK版本和jar的导出版本不一致，52.0是jdk8的版本号
    - 修改Hadoop的hadoop-env.sh，将jdk7的路径修改为jdk8即可

     

