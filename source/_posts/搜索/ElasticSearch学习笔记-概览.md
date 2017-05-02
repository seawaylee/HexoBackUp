---
title: ElasticSearch学习笔记 - 概览
date: 2017-03-09 11:59:15
tags: [搜索]
---

## 1 Introduction
Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。
不过，Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：

- 分布式的实时文件存储，每个字段都被索引并可被搜索
- 分布式的实时分析搜索引擎
- 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

<!--more-->

## 2 Installation
安装ES
```
curl -L -O http://download.elasticsearch.org/PATH/TO/VERSION.zip <1>
unzip elasticsearch-$VERSION.zip
cd  elasticsearch-$VERSION
```
安装Mavel
Marvel是Elasticsearch的管理和监控工具，在开发环境下免费使用。它包含了一个叫做Sense的交互式控制台，使用户方便的通过浏览器直接与Elasticsearch进行交互。

安装Kibana

## 3 API
**Java API**

Elasticsearch为Java用户提供了两种内置客户端：
**节点客户端(node client)：**

节点客户端以无数据节点(none data node)身份加入集群，换言之，它自己不存储任何数据，但是它知道数据在集群中的具体位置，并且能够直接转发请求到对应的节点上。
**传输客户端(Transport client)：**

这个更轻量的传输客户端能够发送请求到远程集群。它自己不加入集群，只是简单转发请求给集群中的节点。
两个Java客户端都通过9300端口与集群交互，使用Elasticsearch传输协议(Elasticsearch Transport Protocol)。集群中的节点之间也通过9300端口进行通信。如果此端口未开放，你的节点将不能组成集群。



**基于HTTP协议，以JSON为数据交互格式的RESTful API**


```shell
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'

```

- VERB HTTP方法：GET, POST, PUT, HEAD, DELETE
- PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）
- HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
- PORT Elasticsearch HTTP服务所在的端口，默认为9200
- PATH API路径（例如_count将返回集群中文档的数量），PATH可以包含多个组件，例如_cluster/stats或者_nodes/stats/jvm
- QUERY_STRING 一些可选的查询请求参数，例如?pretty参数将使请求返回更加美观易读的JSON数据
- BODY 一个JSON格式的请求主体（如果请求需要的话）

`$ curl http://localhost:9200/_count\?pretty`
```json
{
  "count" : 1,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  }
}
```


``$ curl http://localhost:9200\?pretty``
```json
{
  "name" : "Fb_2umy",
  "cluster_name" : "rhino-application",
  "cluster_uuid" : "E1E1h5v5SNGkuMn--DgcPQ",
  "version" : {
    "number" : "5.2.0",
    "build_hash" : "24e05b9",
    "build_date" : "2017-01-24T19:52:35.800Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 4 文档
Elasticsearch是面向文档(document oriented)的，这意味着它可以存储整个对象或文档(document)。然而它不仅仅是存储，还会索引(index)每个文档的内容使之可以被搜索。在Elasticsearch中，你可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。这种理解数据的方式与以往完全不同，这也是Elasticsearch能够执行复杂的全文搜索的原因之一。


**开始第一步**

我们现在开始进行一个简单教程，它涵盖了一些基本的概念介绍，比如**索引(indexing)、搜索(search)以及聚合(aggregations)**。通过这个教程，我们可以让你对Elasticsearch能做的事以及其易用程度有一个大致的感觉。
我们接下来将陆续介绍一些术语和基本的概念，但就算你没有马上完全理解也没有关系。我们将在本书的各个章节中更加深入的探讨这些内容。
所以，坐下来，开始以旋风般的速度来感受Elasticsearch的能力吧！
让我们建立一个员工目录

假设我们刚好在**Megacorp**工作，这时人力资源部门出于某种目的需要让我们创建一个**员工目录**，这个目录用于促进人文关怀和用于实时协同工作，所以它有以下不同的需求：

- 数据能够包含多个值的标签、数字和纯文本。
- 检索任何员工的所有信息。
- 支持结构化搜索，例如查找30岁以上的员工。
- 支持简单的全文搜索和更复杂的短语(phrase)搜索
- 高亮搜索结果中的关键字
- 能够利用图表管理分析这些数据
- 索引员工文档

我们首先要做的是存储员工数据，每个文档代表一个员工。在Elasticsearch中存储数据的行为就叫做索引(indexing)，不过在索引之前，我们需要明确数据应该存储在哪里。
在Elasticsearch中，文档归属于一种类型(type),而这些类型存在于索引(index)中，我们可以画一些简单的对比图来类比传统关系型数据库：

**Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields**
Elasticsearch集群可以包含多个索引(indices)（数据库），每一个索引可以包含多个类型(types)（表），每一个类型包含多个文档(documents)（行），然后每个文档包含多个字段(Fields)（列）。
「索引」含义的区分

`你可能已经注意到索引(index)这个词在Elasticsearch中有着不同的含义，所以有必要在此做一下区分:
索引（名词） 如上文所述，一个索引(index)就像是传统关系数据库中的数据库，它是相关文档存储的地方，index的复数是indices 或indexes。
索引（动词） 「索引一个文档」表示把一个文档存储到索引（名词）里，以便它可以被检索或者查询。这很像SQL中的INSERT关键字，差别是，如果文档已经存在，新的文档将覆盖旧的文档。
倒排索引 传统数据库为特定列增加一个索引，例如B-Tree索引来加速检索。Elasticsearch和Lucene使用一种叫做倒排索引(inverted index)的数据结构来达到相同目的。`

默认情况下，文档中的所有字段都会被索引（拥有一个倒排索引），只有这样他们才是可被搜索的。
我们将会在倒排索引章节中更详细的讨论。
所以为了创建员工目录，我们将进行如下操作：
为每个员工的文档(document)建立索引，每个文档包含了相应员工的所有信息。
每个文档的类型为employee。
employee类型归属于索引megacorp。
megacorp索引存储在Elasticsearch集群中。
实际上这些都是很容易的（尽管看起来有许多步骤）。我们能通过一个命令执行完成的操作：

```json
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

```
请求实体（JSON文档），包含了这个员工的所有信息。他的名字叫“John Smith”，25岁，喜欢攀岩。
很简单吧！它不需要你做额外的管理操作，比如创建索引或者定义每个字段的数据类型。我们能够直接索引文档，Elasticsearch已经内置所有的缺省设置，所有管理操作都是透明的。
接下来，让我们在目录中加入更多员工信息：

```json
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```

## 5 检索文档
我们通过HTTP方法GET来检索文档，同样的，我们可以使用DELETE方法删除文档，使用HEAD方法检查某文档是否存在。如果想更新已存在的文档，我们只需再PUT一次。

```json

GET /megacorp/employee/3
{
  "_index": "megacrop",
  "_type": "employee",
  "_id": "3",
  "_version": 1,
  "found": true,
  "_source": {
    "first_name": "Douglas",
    "last_name": "Fir",
    "age": 35,
    "about": "I like to build cabinets",
    "interests": [
      "forestry"
    ]
  }
}
```

**简单搜索**

GET请求非常简单——你能轻松获取你想要的文档。让我们来进一步尝试一些东西，比如简单的搜索！
我们尝试一个最简单的搜索全部员工的请求：
GET /megacorp/employee/_search
你可以看到我们依然使用megacorp索引和employee类型，但是我们在结尾使用关键字_search来取代原来的文档ID。响应内容的hits数组中包含了我们所有的三个文档。默认情况下搜索会返回前10个结果。+



```json
GET /megacorp/employee/_search

{
  "took": 20,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1.0,
    "hits": [
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "2",
        "_score": 1.0,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      },
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      },
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "3",
        "_score": 1.0,
        "_source": {
          "first_name": "Douglas",
          "last_name": "Fir",
          "age": 35,
          "about": "I like to build cabinets",
          "interests": [
            "forestry"
          ]
        }
      }
    ]
  }
}
```

**加入检索条件**
我们在请求中依旧使用_search关键字，然后将查询语句传递给参数q=。这样就可以得到所有姓氏为Smith的结果：

```json
GET /megacorp/employee/_search?q=last_name:Smith
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      },
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}
```

**使用DSL语句查询**
查询字符串搜索便于通过命令行完成特定(ad hoc)的搜索，但是它也有局限性（参阅简单搜索章节）。Elasticsearch提供丰富且灵活的查询语言叫做DSL查询(Query DSL),它允许你构建更加复杂、强大的查询。
DSL(Domain Specific Language特定领域语言)以JSON请求体的形式出现。我们可以这样表示之前关于“Smith”的查询:

```json
POST /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "Smith"
    }
  }
}
```

**更复杂的搜索**
我们让搜索稍微再变的复杂一些。我们依旧想要找到姓氏为“Smith”的员工，但是我们只想得到年龄大于30岁的员工。我们的语句将添加过滤器(filter),它使得我们高效率的执行一个结构化搜索：

```json
POST /megacorp/employee/_search

{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "last_name": "smith"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "age": {
              "gte": "30"
            }
          }
        }
      ]
    }
  }
}
```

**全文搜索**

```json
POST /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.53484553,
    "hits": [
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "1",
        "_score": 0.53484553,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      },
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "2",
        "_score": 0.26742277,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      }
    ]
  }
}
```

默认情况下，Elasticsearch根据结果相关性评分来对结果集进行排序，所谓的「结果相关性评分」就是文档与查询条件的匹配程度。很显然，排名第一的John Smith的about字段明确的写到“rock climbing”。
但是为什么Jane Smith也会出现在结果里呢？原因是“rock”在她的abuot字段中被提及了。因为只有“rock”被提及而“climbing”没有，所以她的_score要低于John。3
这个例子很好的解释了Elasticsearch如何在各种文本字段中进行全文搜索，并且返回相关性最大的结果集。相关性(relevance)的概念在Elasticsearch中非常重要，而这个概念在传统关系型数据库中是不可想象的，因为传统数据库对记录的查询只有匹配或者不匹配。

**短语搜索**

目前我们可以在字段中搜索单独的一个词，这挺好的，但是有时候你想要确切的匹配若干个单词或者短语(phrases)。例如我们想要查询同时包含"rock"和"climbing"（并且是相邻的）的员工记录。
要做到这个，我们只要将match查询变更为match_phrase查询即可:


```json
POST /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

**高亮搜搜**


```json
POST /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}

```


```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.53484553,
    "hits": [
      {
        "_index": "megacrop",
        "_type": "employee",
        "_id": "1",
        "_score": 0.53484553,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        },
        "highlight": {
          "about": [
            "I love to go <em>rock</em> <em>climbing</em>"
          ]
        }
      }
    ]
  }
}
```

## 6 聚合
最后，我们还有一个需求需要完成：允许管理者在职员目录中进行一些分析。 Elasticsearch有一个功能叫做聚合(aggregations)，它允许你在数据上生成复杂的分析统计。它很像SQL中的GROUP BY但是功能更强大。
举个例子，让我们找到所有职员中最大的共同点（兴趣爱好）是什么：

```json
POST megacrop/employee/_search
{
  "aggregations": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}

```
如果报错


```json
 {
   "type": "illegal_argument_exception",
   "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [interests] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory."
 }

```
则配置属性


```json
POST megacrop/_mapping/employee
{
  "properties": {
    "interests": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
```


返回聚合结果


```json

  "aggregations": {
    "all_interests": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "music",
          "doc_count": 2
        },
        {
          "key": "forestry",
          "doc_count": 1
        },
        {
          "key": "sports",
          "doc_count": 1
        }
      ]
    }
  }
  
```


**嵌套聚合查询**

```json
POST /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```


```json
 "aggregations": {
    "all_interests": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "music",
          "doc_count": 2,
          "avg_age": {
            "value": 28.5
          }
        },
        {
          "key": "forestry",
          "doc_count": 1,
          "avg_age": {
            "value": 35.0
          }
        },
        {
          "key": "sports",
          "doc_count": 1,
          "avg_age": {
            "value": 25.0
          }
        }
      ]
    }
  }

```

## 7 分布式特性简介
在章节的开始我们提到Elasticsearch可以扩展到上百（甚至上千）的服务器来处理PB级的数据。然而我们的教程只是给出了一些使用Elasticsearch的例子，并未涉及相关机制。Elasticsearch为分布式而生，而且它的设计隐藏了分布式本身的复杂性。
Elasticsearch在分布式概念上做了很大程度上的透明化，在教程中你不需要知道任何关于分布式系统、分片、集群发现或者其他大量的分布式概念。所有的教程你既可以运行在你的笔记本上，也可以运行在拥有100个节点的集群上，其工作方式是一样的。
Elasticsearch致力于隐藏分布式系统的复杂性。以下这些操作都是在底层自动完成的：
将你的文档分区到不同的容器或者分片(shards)中，它们可以存在于一个或多个节点中。
将分片均匀的分配到各个节点，对索引和搜索做负载均衡。
冗余每一个分片，防止硬件故障造成的数据丢失。
将集群中任意一个节点上的请求路由到相应数据所在的节点。
无论是增加节点，还是移除节点，分片都可以做到无缝的扩展和迁移。
## 8 小结
现在你对Elasticsearch可以做些什么以及其易用程度有了大概的了解。Elasticsearch致力于降低学习成本和轻松配置。学习Elasticsearch最好的方式就是开始使用它：开始索引和检索吧！
当然，你越是了解Elasticsearch，你的生产力就越高。你越是详细告诉Elasticsearch你的应用的数据特点，你就越能得到准确的输出。
本书其余部分将帮助你从新手晋级到专家。每一个章节都会阐述一个要点，并且会包含专家级别的技巧。如果你只是刚起步，那么这些技巧可能暂时和你无关。Elasticsearch有合理的默认配置而且可以在没有用户干预的情况下做正确的事情。当需要提升性能时你可以随时回顾这些章节。

