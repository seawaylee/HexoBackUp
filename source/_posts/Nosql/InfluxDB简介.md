---
title: InfluxDB学习笔记（一）- InfluxDB简介
date: 2017-06-12 12:36:28
comments: true
tags: [Nosql]
---


## 1 简介
[InfluxDB](https://docs.influxdata.com/influxdb/v1.2/introduction/getting_started/) 是一个开源分布式时序、事件和指标数据库。使用 Go 语言编写，无需外部依赖。其设计目标是实现分布式和水平伸缩扩展。
它有三大特性：

1. Time Series （时间序列）：你可以使用与时间有关的相关函数（如最大，最小，求和等）
2. Metrics（度量）：你可以实时对大量数据进行计算
3. Eevents（事件）：它支持任意的事件数据

**特点**

* schemaless(无结构)，可以是任意数量的列
* Scalable
* min, max, sum, count, mean, median 一系列函数，方便统计
* Native HTTP API, 内置http支持，使用http读写
* Powerful Query Language 类似sql
* Built-in Explorer 自带管理工具


<!--more-->

## 2 CLI(Command Line Interface)操作

### 2.1 安装 启动

**Install:**  `brew install influxdb`
**Start:**  `influxd run`
**Connect:**  `influx`

### 2.2 CLI

* 帮助信息 


```
Usage:
        connect <host:port>   connects to another node specified by host:port
        auth                  prompts for username and password
        pretty                toggles pretty print for the json format
        use <db_name>         sets current database
        format <format>       specifies the format of the server responses: json, csv, or column
        precision <format>    specifies the format of the timestamp: rfc3339, h, m, s, ms, u or ns
        consistency <level>   sets write consistency level: any, one, quorum, or all
        history               displays command history
        settings              outputs the current settings for the shell
        clear                 clears settings such as database or retention policy.  run 'clear' for help
        exit/quit/ctrl+d      quits the influx shell

        show databases        show database names
        show series           show series information
        show measurements     show measurement information
        show tag keys         show tag key information
        show field keys       show field key information

        A full list of influxql commands can be found at:
        https://docs.influxdata.com/influxdb/latest/query_language/spec/
```

* 创建数据库 `create database rhinotech` 
* 显示所有DB  `show databases`

```commandline
> show databases
name: databases
name
----
_internal
rhinotech
```

* 使用数据库 `use rhinotech`

* [写数据](https://docs.influxdata.com/influxdb/v1.2/tools/shell/) `insert students,name=NikoBelic,age=18,region=china value=888,shit=999`

>Now that we have a database, InfluxDB is ready to accept queries and writes. First, a short primer on the datastore. Data in InfluxDB is organized by “**time series**”, which contain a measured value, like “cpu_load” or “temperature”. Time series have zero to many points, one for each discrete sample of the metric. Points consist of **time** (a timestamp), a **measurement** (“cpu_load”, for example), at least one key-value field (the measured value itself, e.g. “value=0.64”, or “temperature=21.2”), and zero to many key-value **tags** containing any metadata about the value (e.g. “host=server01”, “region=EMEA”, “dc=Frankfurt”). Conceptually you can think of a** measurement as an SQL table**, where the **primary index is always time**. **tags and fields are effectively columns in the table**. tags are indexed, and fields are not. The difference is that, with InfluxDB, you can have millions of measurements, you don’t have to define schemas up-front, and null values aren’t stored.

时序库与关系库相关名词对应：

| 时序库 | 关系库 | 备注 |
| --- | --- | --- |
| time | id | 主键 |
| measurement | table | 表 |
| tags | column + index| 带索引的列 |
| fields | column| 普通列 |

tags和fields差不多，tags自带索引，因此查询时候更快。

* [查询数](https://docs.influxdata.com/influxdb/v1.2/guides/querying_data/) `select * from students`


```commandline
> select * from students
name: students
time                age name      region shit value
----                --- ----      ------ ---- -----
1494490902239708732 18  NikoBelic china       888
1494492099547889163 18  NikoBelic china  999  888
```

## 3 HTTP 读写数据

### 3.1 [Writing Data With Http API](https://docs.influxdata.com/influxdb/v1.2/guides/writing_data/)

* 创建数据库 `curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"`
* 写数据库  `curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu_load_info,host=server1,region=A3-1-1 load=0.64'`
* 从文件导入数据

文件格式

```
cpu_load_short, host=server02 value=0.67
cpu_load_short, host=server02, region=us-west value=0.55 1422568543702900257
cpu_load_short, direction=in,host=server01,region=us-west value=2.0 1422568543702900257
```
导入 `curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary @cpu_data.txt`

注意 如果文件中的数据数量超过5000个，建议切分成多个文件批量导入，因为http请求的默认超时时间为5s，虽然请求超时后influxdb仍然会继续导入数据，但是客户端将无法收到导入成功的提示信息。

### 3.2 Querying Data With Http API

* 查询 `curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=SELECT * FROM \"cpu_load_info\""`

结果


```json
{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "name": "cpu_load_info",
                    "columns": [
                        "time",
                        "host",
                        "load",
                        "region",
                        "size"
                    ],
                    "values": [
                        [
                            "2017-05-13T05:28:52.323145124Z",
                            "server1",
                            0.64,
                            "A3-1-1",
                            null
                        ],
                        [
                            "2017-05-13T05:55:47.007680205Z",
                            null,
                            0.64,
                            "A3-1-1",
                            "120"
                        ]
                    ]
                }
            ]
        }
    ]
}
```

使用 `--data-urlencode "epoch=h"` 可以控制显示的时间戳格式[h,m,s,ms,u,ns]

