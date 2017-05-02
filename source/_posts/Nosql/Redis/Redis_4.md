---
title: Redis学习（四） 数据安全与性能 
date: 2017-03-09 21:50
tags: [Nosql,Redis]
---

## 4.1 持久化选项
持久化有两种方式：

- 快照 snapshot：将某一时刻的所有数据都写入到硬盘里面
- 只追加文件 append-only file：执行写命令时，将写命令复制到硬盘里面。

两种持久化方式可以同时或单独或都不使用。

<!--more-->

**基本命令**

- Snapshot
    - save 60 1000
    - stop-writes-on-bgsave-error no
    - rdbcompression yes
    - dbfilename dump.rdb

- append-only file
    - appendonly no
    - appendsync everysec
    - no-appendfsync-on-rewrite no
    - auto-aof-rewrite-percentage 100
    - auto-aof-rewrite-min-size 64mb

- dir ./ 共享选项，决定了snapshot和AOF文件的保存位置。




### 4.1.1 快照持久化

创建snapshot的方法

- BGSAVE命令创建快照。Redis会调用fork来创建一个子进程来将快照写入硬盘。
- SAVE命令创建快照，阻塞式写入硬盘。
- save 60 1000 从Redis最近一次创建快照之后算起，“60秒内有1000次写入”条件满足时自动触发BGSAVE。如果有多个save配置，当任意一个配置满足时将触发BGSAVE。
- Redis收到系统SHUTDOWN命令时，或标准TERM信号时，自动执行SAVE，阻塞所有客户端。完成后关闭服务器。
- 当一个Redis连接另一个Redis，并向对方发送SYNC命令开始复制时，如果主服务器目前没有BGSAVE执行，或并非刚刚完成BGSAVE，则主服务器自动执行BGSAVE。

### 4.1.2 AOF持久化
AOP持久化会将被执行的写命令写到AOF文件的末尾，以此来记录数据发生的变化。

- 使用aooendonly yes配置选项打开AOF持久化
- 参数
    - always: 每个Redis写命令都同步到硬盘，严重降低性能。
    - everysec：每秒执行以此同步，显示地将多个写命令同步到硬盘
    - no：让操作系统来决定该合适进行同步
- 使用auto-aof-rewrite-percentage 100 配置AOF文件体积达到指定百分比时进行重写（子进程）
- 使用auto-aof-rewrite-min-size 64mb 配置AOF文件大于64MB时进行重写（子进程）

## 4.2 复制
A：主服务器，B：从服务器

- A:Server Start
- B:Server Start,Cli Start,slaveof IP(A) PORT(A)

主从复制时步骤

|步骤  | 主服务器操作 | 从服务器操作 |
| --- | --- | --- |
| 1 | 等待命令 | 连接主服务器，发送SYNC命令 |
| 2 | 执行BGSAVE，使用缓冲区记录BGSAVE之后执行的所有写命令 |根据config来决定继续使用现有数据来返回给客户端请求，还是返回错误 |
| 3 | BGSAVE执行完毕，向从服务器发送snapshot,使用缓冲区记录期间的写命令 | 丢弃旧数据，载入收到的snapshot文件 |
| 4 | snapshot发送完毕，向从服务器发送缓冲区中的写命令 | 正常接受命令请求 |
| 5 | 缓冲区命令发送完毕，每执行一个写命令，就向从服务器发送相同的写命令 | 执行主服务器发来的缓冲区写命令，并从现在开始接收并执行主服务器传来的每个写命令 |



当需要更换主服务器时，可以直接将主服务器的dump文件复制到新的主服务器，从服务器重新指定新的主服务器。

## 4.3 Redis事务

通过使用WATCH、MULTI/EXEC、UNWACH/DISCARD等命令，程序可以在执行某些重要操作的时候，通过确保自己正在使用的数据没有发生变化来避免数据出错。

一般执行流程如下

```commandline
conn.pipeline()
pipe.watch(item)
pipe.multi()
pipe.zadd()
pipe.srem()
pipe.execute()
```
## 4.4 非事务型流水线

MULTI和EXEC可以让程序拥有事务性，有些情况下，我们不需要事务来将操作回滚，只是想减少应用和Redis之间的通讯次数。创建pipe的时候，传入False参数即可。

```commandline
conn.pipeline(False)
pipe.multi()
pipe.zadd()
pipe.srem()
pipe.execute()
```

