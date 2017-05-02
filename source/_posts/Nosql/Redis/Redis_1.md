---
title: Redis学习（一）数据结构
date: 2017-03-09 21:38:17
tags: [Nosql,Redis]
---


## 1 Redis数据结构简介

| 结构类型 | 结构存储的值 | 结构的读写能力 |
| --- | --- | --- |
| String | 字符串、整数、浮点数  | 对字符串或字符串的一部分执行操作；对整数、浮点数进行自增、自减 |
| List | 一个链表，链表上的每个节点都包含了一个字符串 | 从链表的两端push或者pop元素，根据index对链表进行trim；读取单个或多个元素；根据值查找或移除元素 |
| Set | 字符串无序collection，每个String独一无二 | 添加、获取、删除单个元素；检查元素是否存在于collection中；计算交集、并集、差集；从集合里随机获取元素 |
| Hash | 包含键值对的无序散列表 | 添加、获取、移除单个键值对；获取所有键值对 |
| ZSet | 字符串成员与浮点数score之间的有序映射，元素的排列顺序由大小决定 | 添加、获取、删除单个元素；根据score的range或者成员来获取元素 |

<!--more-->

### 1.1 String

```commandline
127.0.0.1:6379[3]> set hello world
OK
127.0.0.1:6379[3]> keys *
1) "hello"
127.0.0.1:6379[3]> get hello
"world"
```

### 1.2 List

| 命令 | 行为 |
| --- | --- |
| rpush | 右端添加 |
| lpush | 左端添加 |
| lindex | 获取列表在给定位置上的单个元素 |
| lpop | 从列表左端弹出一个值，并返回被弹出的值 |

push操作后会返回当前队列的长度
pop操作后会返回被弹出的元素
lindex操作会返回指定位置的值（注意index从0开始）
lrange start end 获取指定范围内的元素，end=-1表示获取直到最后一个元素

```commandline
127.0.0.1:6379[3]> lpush mylist a
(integer) 1
127.0.0.1:6379[3]> lpush mylist b
(integer) 2
127.0.0.1:6379[3]> lrange mylist 0 -1
1) "b"
2) "a"
127.0.0.1:6379[3]> rpush mylist c
(integer) 3
127.0.0.1:6379[3]> lrange mylist 0 -1
1) "b"
2) "a"
3) "c"
127.0.0.1:6379[3]> lpop mylist
"b"
127.0.0.1:6379[3]> lrange mylist 0 -1
1) "a"
2) "c"
127.0.0.1:6379[3]> lindex mylist 2
(nil)
127.0.0.1:6379[3]> lindex mylist 1
"c"
127.0.0.1:6379[3]> lindex mylist 0
"a"
```



### 1.3 Set集合


| 命令 | 行为 |
| --- | --- |
| sadd | 将元素添加到集合 |
| smembers | 返回集合包含的所有元素 |
| sismember | 检查给定元素是否存在与集合中 |
| srem | 如果给定的元素存在于集合中，那么移除这个元素 |

```commandline
127.0.0.1:6379[3]> sadd myset NikoBelic
(integer) 1
127.0.0.1:6379[3]> SMEMBERS myset
1) "NikoBelic"
127.0.0.1:6379[3]> sadd myset Tom
(integer) 1
127.0.0.1:6379[3]> sadd myset Helen
(integer) 1
127.0.0.1:6379[3]> sadd myset Marry
(integer) 1
127.0.0.1:6379[3]> SMEMBERS myset
1) "Marry"
2) "Tom"
3) "Helen"
4) "NikoBelic"
127.0.0.1:6379[3]> SISMEMBER myset Tom
(integer) 1
127.0.0.1:6379[3]> SISMEMBER myset Tom222
(integer) 0
127.0.0.1:6379[3]> srem myset NikoBelic
(integer) 1
127.0.0.1:6379[3]> srem myset NikoBelic222
(integer) 0
127.0.0.1:6379[3]> SMEMBERS myset
1) "Marry"
2) "Tom"
3) "Helen"
```

### 1.4 散列

Redis的散列更像是一个猥琐扮的memcache数据库

| 命令 | 行为 |
| --- | --- |
| hset | 在散列里面关联起给定的键值对 |
| hget | 获取指定散列键的值 |
| hgetall | 获取散列包含的所有键值对 |
| hdel | 如果给定键存在于散列里面，那么移除这个键 |


```commandline
127.0.0.1:6379[3]> hset myhash name NikoBelic
(integer) 1
127.0.0.1:6379[3]> hset myhash age 18
(integer) 1
127.0.0.1:6379[3]> hset myhash address BeiJing
(integer) 1
127.0.0.1:6379[3]> HGETALL myhash
1) "name"
2) "NikoBelic"
3) "age"
4) "18"
5) "address"
6) "BeiJing"
127.0.0.1:6379[3]> hget myhash name
"NikoBelic"
127.0.0.1:6379[3]> hdel myhash name
(integer) 1
127.0.0.1:6379[3]> hdel myhash age
(integer) 1
127.0.0.1:6379[3]> hgetall myhash
1) "address"
2) "BeiJing"
```

### 1.5 有序集合
| 命令 | 行为 |
| --- | --- |
| zadd | 将一个带有给定分值的成员添加到有序集合里面 |
| zrange | 根据元素在有序排列中所处的位置，从有序集合里面获取多个元素 |
| zrangebyscore | 获取有序集合在给定分值范围内的所有元素 |
| zrem | 如果给定成员存在于有序集合，那么移除这个元素 |




```commandline
127.0.0.1:6379[3]> zadd myorderset 100 NikoBelic
(integer) 1
127.0.0.1:6379[3]> zadd myorderset 101 Tom
(integer) 1
127.0.0.1:6379[3]> zadd myorderset 102 Helen
(integer) 1
127.0.0.1:6379[3]> zadd myorderset 10 Marry
(integer) 1
127.0.0.1:6379[3]> zrange myorderset 0 -1
1) "Marry"
2) "NikoBelic"
3) "Tom"
4) "Helen"
127.0.0.1:6379[3]> zrange myorderset 0 -1 withscores
1) "Marry"
2) "10"
3) "NikoBelic"
4) "100"
5) "Tom"
6) "101"
7) "Helen"
8) "102"
127.0.0.1:6379[3]> ZRANGEBYSCORE myorderset 100 103 withscores
1) "NikoBelic"
2) "100"
3) "Tom"
4) "101"
5) "Helen"
6) "102"
```

## 小结
以上是对Redis的5中数据结果的基本操作进行的简要概述，还有一些其他操作，例如自增(INCR)等操作以后会说，但以上都是日常开发中最最常用的操作，需要熟记于心。

