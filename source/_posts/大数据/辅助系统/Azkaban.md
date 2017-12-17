---
title: 大数据架构辅助系统（二）- Azkaban学习笔记
date: 2017-10-08 19:24:28
tags: [大数据]
toc: true
---


## 1 工作流调度器
### 1.1 为什么需要工作流调度系统

* 一个完整的数据分析系统通常都是由大量任务单元组成：
    * shell脚本程序
    * java程序
    * mapreduce程序
    * hive脚本等、
* 各任务单元之间存在时间先后及前后依赖关系
* 为了很好地组织起这样的复杂执行计划，需要一个工作流调度系统来调度执行

例如，我们可能有这样一个需求，某个业务系统每天产生20G原始数据，我们每天都要对其进行处理，处理步骤如下所示：

1. 通过Hadoop先将**原始数据同步到HDFS**上；
2. 借助MapReduce计算框架对**原始数据进行转换**，生成的数据以分区表的形式**存储到多张Hive表**中；
3. 需要对Hive中多个**表的数据进行JOIN**处理，得到一个**明细数据Hive大表**；
4. 将明细数据进行**复杂的统计分析**，得到结果报表信息；
5. 需要将统计分析得到的结果数据**同步到业务系统**中，供业务调用使用。

<!--more-->

### 1.2 工作流调度实现方式

* 简单的任务调度：直接使用linux的crontab来定义；
* 复杂的任务调度：
    * 开发调度平台
    * 使用现成的开源调度系统，比如ooize、azkaban等


### 1.3 常见工作流调度系统

市面上目前有许多工作流调度器
在hadoop领域，常见的工作流调度器有**Oozie**, **Azkaban**,**Cascading**,**Hamake**等

**各种调度工具特性对比**

下面的表格对上述四种hadoop工作流调度器的关键特性进行了比较，尽管这些工作流调度器能够解决的需求场景基本一致，但在设计理念，目标用户，应用场景等方面还是存在显著的区别，在做技术选型的时候，可以提供参考

![](15072764100819.jpg)


## 2 Azkaban

>Azkaban是由Linkedin开源的一个批量工作流任务调度器。用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban定义了一种**KV文件格式**来建立任务之间的依赖关系，并提供一个易于使用的**web用户界面**维护和跟踪你的工作流。

它有如下功能特点：

* Web用户界面
* 方便上传工作流
* 方便设置任务之间的关系
* 调度工作流
* 认证/授权(权限的工作)
* 能够杀死并重新启动工作流
* 模块化和可插拔的插件机制
* 项目工作区
* 工作流和任务的日志记录和审计



### 2.1 安装和部署

**Azkaban Web服务器**
azkaban-web-server-2.5.0.tar.gz

**Azkaban执行服务器 **
azkaban-executor-server-2.5.0.tar.gz

下载地址:http://azkaban.github.io/downloads.html

**MySQL**
目前azkaban只支持 mysql,需安装mysql服务器

 
**安装**

- 将安装文件上传到集群,最好上传到安装 hive、sqoop的机器上,方便命令的执行
- 在当前用户目录下新建 azkabantools目录,用于存放源安装文件.新建azkaban目录,用于存放azkaban运行程序
- azkaban web服务器安装
    1. 解压azkaban-web-server-2.5.0.tar.gz
    2. 命令: tar –zxvf azkaban-web-server-2.5.0.tar.gz
    3. 将解压后的azkaban-web-server-2.5.0 移动到 azkaban目录中,并重新命名 webserver
- azkaban 执行服器安装
    1. 解压azkaban-executor-server-2.5.0.tar.gz
    2. 命令:tar –zxvf azkaban-executor-server-2.5.0.tar.gz
    3. 将解压后的azkaban-executor-server-2.5.0 移动到 azkaban目录中,并重新命名 executor
- azkaban脚本导入
    1. 解压: azkaban-sql-script-2.5.0.tar.gz
    2. 命令:tar –zxvf azkaban-sql-script-2.5.0.tar.gz
    3. 将解压后的mysql 脚本,导入到mysql中:
    4. 进入mysql
        * mysql> create database azkaban;
        * mysql> use azkaban;
        * Database changed
        * mysql> source /home/hadoop/azkaban-2.5.0/create-all-sql-2.5.0.sql;

**由于Azkaban的web管理页面需要HTTPS访问，所以需要配置HTTPS服务**

**SSL配置**

* 参考地址: http://docs.codehaus.org/display/JETTY/How+to+configure+SSL
* 命令: keytool -keystore keystore -alias jetty -genkey -keyalg RSA
* 将在当前目录生成 keystore 证书文件,将keystore 拷贝到 azkaban web服务器根目录中.如:cp keystore azkaban/server

**启动Web服务**

`./bin/azkaban-web-start.sh`

![-w1280](15073737056510.jpg)

**启动Executor**

`./bin/azkaban-executor-start.sh`

### 2.2 Azkaban实战

>Azkaba内置的任务类型支持command、java

#### 2.2.1 Command类型单一job示例

- 创建job描述文件
- vi command.job

```commnad
#command.job
type=command                                                    
command=echo 'hello'
```

- 将job资源文件打包成zip文件 `zip command.job`
- 通过azkaban的web管理平台创建project并上传job压缩包
    - 创建project
    - 上传zip任务包
    - 执行job

![](15074469180070.jpg)
![](15074469243060.jpg)
![](15074469334556.jpg)



#### 2.2.2 Command类型多Job工作流Flow

>创建有依赖关系的多个job描述

- 第一个job：foo.job

```cmd
# foo.job
type=command
command=echo foo
```

- 第二个job：bar.job依赖foo.job

```cmd
# bar.job
type=command
dependencies=foo
command=echo bar
```

- 将所有job资源文件打到一个zip包中

![](15074470375415.jpg)

- 在azkaban的web管理界面创建工程并上传zip包
- 启动工作流flow

#### 2.2.3 HDFS操作任务

1. 创建job描述文件


```cmd
# fs.job
type=command
command=/home/hadoop/apps/hadoop-2.6.1/bin/hadoop fs -mkdir /azaz
```

2. 将job资源文件打包成zip文件
![](15074479436571.jpg)
    

3. 通过azkaban的web管理平台创建project并上传job压缩包
4. 启动执行该job


#### 2.2.3 MAPREDUCE任务

>Mr任务依然可以使用command的job类型来执行

1. 创建job描述文件，及mr程序jar包（示例中直接使用hadoop自带的example jar）

```cmd
# mrwc.job
type=command
command=/home/hadoop/apps/hadoop-2.6.1/bin/hadoop  jar hadoop-mapreduce-examples-2.6.1.jar wordcount /wordcount/input /wordcount/azout
```

2. 将所有job资源文件打到一个zip包中
3. 在azkaban的web管理界面创建工程并上传zip包
4. 启动job

#### 2.2.4 HIVE脚本任务

1. 创建job描述文件和hive脚本  

Hive脚本： test.sql

```sql
use default;
drop table aztest;
create table aztest(id int,name string) row format delimited fields terminated by ',';
load data inpath '/aztest/hiveinput' into table aztest;
create table azres as select * from aztest;
insert overwrite directory '/aztest/hiveoutput' select count(1) from aztest; 
```

Job描述文件：hivef.job

```cmd
# hivef.job
type=command
command=/home/hadoop/apps/hive/bin/hive -f 'test.sql'
```

2. 将所有job资源文件打到一个zip包中
3. 在azkaban的web管理界面创建工程并上传zip包
4. 启动job

