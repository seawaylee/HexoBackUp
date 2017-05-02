---
title: Redis学习（二）数据结构进阶
date: 2017-03-09 21:48
tags: [Nosql,Redis]
---

## 2.1 字符串

**Redis处理自增自减的命令**

| 命令 | 用例和描述 |
| --- | --- |
| incr | incr keyname   值+1 |
| decr | decr keyname   值-1 |
| incrby | incrby keyname amount  值+amount |
| decrby | decrby keyname amount  值-amount |
| incrbyfloat | incrbyfloat keyname amount  值+float类型的amount(after redis 2.6) |

<!--more-->


```commandline
127.0.0.1:6379[3]> set mykey 1
OK
127.0.0.1:6379[3]> get mykey
"1"
127.0.0.1:6379[3]> incr mykey
(integer) 2
127.0.0.1:6379[3]> incr mykey
(integer) 3
127.0.0.1:6379[3]> get mykey
"3"
127.0.0.1:6379[3]> set mystr abc
OK
127.0.0.1:6379[3]> incr mystr
(error) ERR value is not an integer or out of range
127.0.0.1:6379[3]> incrby mykey 100
(integer) 103
127.0.0.1:6379[3]> decr mykey
(integer) 102
127.0.0.1:6379[3]> decrby mykey 50
(integer) 52
127.0.0.1:6379[3]> INCRBYFLOAT mykey 10
"62"
127.0.0.1:6379[3]> get mykey
"62"
```

Redis处理子串和二进制位的命令

| 命令 | 用例和描述 |
| --- | --- |
| append | append keyname value 将值value追加到keyname当前值的末尾 |
| getrange | getrange keyname start end 截取字符串[start,end] |
| setrange | setrange keyname offset value 将从start开始的子串设置为给定值 |
| getbit | getbit keyname offset 将字节串看做是二进制位串，并返回位串中偏移量为offset的二进制位的值 |
| setbit | setbit keyname offset value 将二进制位串offset位置的值设置为value |
| bitcount | bitcount keyname [stard end] 统计二进制位串里面value=1的数量，如果给定了start和end，则统计该范围内的 |
| bitop | bitop operation destkey keyname [keyname…] 对一个或多个二进制位串执行包括 and、or、xor、not在内的人呢以一种按位运算操作(operation)，并将计算得到的记过保存在destkey中 |



```commandline
127.0.0.1:6379[3]> set mystr 'Hello!! My name is NikoBelic'
OK
127.0.0.1:6379[3]> get mystr
"Hello!! My name is NikoBelic"
127.0.0.1:6379[3]> append mystr ", 18 years old"
(integer) 42
127.0.0.1:6379[3]> get mystr
"Hello!! My name is NikoBelic, 18 years old"
127.0.0.1:6379[3]> setrange mystr 0 Hey~~!!
(integer) 42
127.0.0.1:6379[3]> get mystr
"Hey~~!! My name is NikoBelic, 18 years old"
```

## 2.2 列表
rpush、lpush、rpop、lpop、lindex、lrange在第一篇文章介绍过，不多bb了。
| 命令 | 用例和描述 |
| --- | --- |
| ltrim  | ltrim keyname start end  对列表进行修剪，只保留[start,end]范围内的元素 |


```commandline
127.0.0.1:6379[3]> lpush mylist NikoBelic
(integer) 3
127.0.0.1:6379[3]> rpush mylist Tom
(integer) 4
127.0.0.1:6379[3]> lrange mylist 0 -1
1) "NikoBelic"
2) "a"
3) "c"
4) "Tom"
127.0.0.1:6379[3]> ltrim mylist 0 3
OK
127.0.0.1:6379[3]> lrange mylist 0 -1
1) "NikoBelic"
2) "a"
3) "c"
4) "Tom"
127.0.0.1:6379[3]> ltrim mylist 0 2
OK
127.0.0.1:6379[3]> lrange mylist 0 -1
1) "NikoBelic"
2) "a"
3) "c"
```

阻塞式列表弹出命令以及在列表之间移动元素命令

| 命令 | 用例和描述 |
| --- | --- |
| blpop | blpop keyname [keyname…] timeout  从第一个非空列表中弹出位于最左端的元素，或者在timeout秒之内阻塞并等待可弹出元素的出现 |
| brpop | ... |
| rpoplpush  | rpoplpush sourcekey destkey  从source列表中弹出位于最右端的元素，然后将这个而元素推入到dest |
| brpoplpush | brpoplpush sourcekey destkey timeout  ….在timeout秒之内阻塞并等待可弹出的元素出现 |



```commandline
127.0.0.1:6379[3]> rpush sourcekey A
(integer) 1
127.0.0.1:6379[3]> rpush sourcekey B
(integer) 2
127.0.0.1:6379[3]> rpush sourcekey C
(integer) 3
127.0.0.1:6379[3]> lrange sourcekey
(error) ERR wrong number of arguments for 'lrange' command
127.0.0.1:6379[3]> lrange sourcekey 0 -1
1) "A"
2) "B"
3) "C"
127.0.0.1:6379[3]> rpoplpush sourcekey destkey
"C"
127.0.0.1:6379[3]> rpoplpush sourcekey destkey
"B"
127.0.0.1:6379[3]> lrange sourcekey 0 -1
1) "A"
127.0.0.1:6379[3]> lrange destkey 0 -1
1) "B"
2) "C"
127.0.0.1:6379[3]> blpop sourcekey 10
1) "sourcekey"
2) "A"
127.0.0.1:6379[3]> blpop sourcekey 10
(nil)
(10.07s)
```

测试多个keyname的blocking弹出，从结果可以看出 blpop多个keyname不是分别将两个list的内容弹出，而是优先弹出第一个key，如果弹出成功则完成，如果第一个key没有内容，弹出操作不会被block，而是到下一个key中尝试弹出，如果弹出成功则结束，如果弹出失败则block timeout秒的时间。说明我们可以使用Redis完成消息的队列的功能。

```commandline
127.0.0.1:6379[3]> blpop sourcekey 10
1) "sourcekey"
2) "A"
127.0.0.1:6379[3]> blpop sourcekey 10
(nil)
(10.07s)
127.0.0.1:6379[3]> lpush sourcekey blockingtest
(integer) 1
127.0.0.1:6379[3]> blpop sourcekey destkey 10
1) "sourcekey"
2) "blockingtest"
127.0.0.1:6379[3]> lrange destkey 0 -1
1) "B"
2) "C"
127.0.0.1:6379[3]> blpop sourcekey destkey 10
1) "destkey"
2) "B"
127.0.0.1:6379[3]> blpop sourcekey destkey 10
1) "destkey"
2) "C"
127.0.0.1:6379[3]> blpop sourcekey destkey 10
(nil)
(10.08s)
```

## 2.3 集合

| 命令 | 用例和描述 |
| --- | --- |
| sadd | sadd keyname item [item…] 将元素添加到集合里面，并返回被添加元素当中原本不存在于集合中的元素数量  |
| srem | srem keyname item [item…] 从集合里面移除一个或多个元素，并返回被移除元素的个数 |
| sismember | sismember keyname 检查item是否存在于集合 |
| scard | scard keyname 返回集合包含元素的数量 |
| smembers | smsmbers keyname 返回集合包含的所有元素 |
| spop | spop keyname 随机的移除一个元素，并返回被移除的元素 |
| smove | smove sourcekey destkey item 如果source中包含item，则将item从source移除并添加到dest中；如果item被成功移除则返回1，否则返回0 |
| srandmember | srandmember keyname [count] 从集合里面随机返回count个元素。当count为正数时，返回的随机元素不会重复；当count为负数时，返回的元素可能出现重复。 |



```commandline
127.0.0.1:6379[3]> SMEMBERS myset
1) "Marry"
2) "Tom"
3) "Helen"
127.0.0.1:6379[3]> sadd myset Marry
(integer) 0
127.0.0.1:6379[3]> sadd myset Marry2
(integer) 1
127.0.0.1:6379[3]> srem myset asd
(integer) 0
127.0.0.1:6379[3]> srem myset Marry2
(integer) 1
127.0.0.1:6379[3]> scard myset
(integer) 3
127.0.0.1:6379[3]> smembers myset
1) "Marry"
2) "Tom"
3) "Helen"
127.0.0.1:6379[3]> spop myset
"Helen"
127.0.0.1:6379[3]> smove myset newset Tom
(integer) 1
127.0.0.1:6379[3]> smembers myset
1) "Marry"
127.0.0.1:6379[3]> smembers newset
1) "Tom"
127.0.0.1:6379[3]> sadd myset Niko
(integer) 1
127.0.0.1:6379[3]> sadd myset Nicholas
(integer) 1
127.0.0.1:6379[3]> sadd myset James
(integer) 1
127.0.0.1:6379[3]> smembers myset
1) "Marry"
2) "Nicholas"
3) "Niko"
4) "James"
127.0.0.1:6379[3]> srandmember myset 3
1) "Marry"
2) "Niko"
3) "Nicholas"
127.0.0.1:6379[3]> srandmember myset -3
1) "Marry"
2) "James"
3) "Marry"
```

**多个集合的操作**


| 命令 | 用例和描述 |
| --- | --- |
| sdiff | sdiff keyname [keyname…] 返回那些存在于第一个结合但不存在于其他集合中的元素（差集） |
| sdiffstore | sdiff destkey keyname [keyname…] 将集合的差集存储在destkey中 |
| sinter | sinter keyname [keyname…] 返回存在于所有集合中的元素（交集） |
| sinterstore | sinterstore destkey keyname [keyname…] 将交集存储于destkey中 |
| sunion | sunion keyname [keyname…] 返回至少存在于一个集合中的元素(并集) |
| sunionscore | sunitonscore destkey keyname [keyname…] 将并集结果存储在destkey中 |



```commandline
127.0.0.1:6379[3]> smembers myset
1) "Marry"
2) "Nicholas"
3) "Niko"
4) "James"
127.0.0.1:6379[3]> smembers newset
1) "Tom"
127.0.0.1:6379[3]> sdiff myset newset
1) "Marry"
2) "Niko"
3) "Nicholas"
4) "James"
127.0.0.1:6379[3]> sinter myset newset
(empty list or set)
127.0.0.1:6379[3]> sunion myset newset
1) "Marry"
2) "Niko"
3) "Nicholas"
4) "James"
5) "Tom"
```

## 2.4 散列

**散列基本操作**

| 命令 | 用例和描述 |
| --- | --- |
| hmget | hmget keyname key [key…] 从散列里面获取一个或多个键的值 |
| hmset | hmset keyname key value [key value…] 为散列里面的键设置值 |
| hdel | hdel keyname key [key…] 删除散列里面的一个或多个键对，并返回成功找到并删除的键值对数量 |
| hlen | hlen keyname 返回散列包含的键值对数量 |

**散列高级操作**

| 命令 | 用例和描述 |
| --- | --- |
| hexists | hexists keyname key 检查给定的键是否存在于散列中 |
| hkeys | hkeys keyname 获取散列包含的所有键 |
| kvals | hvals keyname 获取散列包含的所有值 |
| hgetall | hgetall keyname 获取散列包含的所有键值对 |
| hincrby | hincrby keyname key increment 将键key存储的值加上整数increment |
| hincrbyfloat | hincrbyfloat keyname key increment 将键key存储的值加上浮点数increment |



```commandline
127.0.0.1:6379[3]> hmset myhash notebook MacBookPro
OK
127.0.0.1:6379[3]> hmget myhash notebook
1) "MacBookPro"
127.0.0.1:6379[3]> hexists myhash notebook
(integer) 1
127.0.0.1:6379[3]> hkeys myhash
1) "address"
2) "notebook"
127.0.0.1:6379[3]> hvals myhash
1) "BeiJing"
2) "MacBookPro"
127.0.0.1:6379[3]> hgetall myhash
1) "address"
2) "BeiJing"
3) "notebook"
4) "MacBookPro"
127.0.0.1:6379[3]> hmset myhash count 1
OK
127.0.0.1:6379[3]> hincr myhash count
(error) ERR unknown command 'hincr'
127.0.0.1:6379[3]> hincrby myhash count
(error) ERR wrong number of arguments for 'hincrby' command
127.0.0.1:6379[3]> hincrby myhash count 5
(integer) 6
127.0.0.1:6379[3]> hmget myhash count
1) "6"
```

## 2.4 有序集合


| 命令 | 用例和描述 |
| --- | --- |
| zadd | zadd keyname score member [score member] 将带有给定分支的成员添加到集合里面 |
| zrem | zrem keyname member [member…]  从有序集合里面移除给定的成员，并返回被移除成员的数据 |
| zcard | zcard keyname  返回有序集合包含的成员数量 |
| zincrby | zincrby keyname increment member 将member的分值加上increment |
| zcount | zcount keyname min max 返回分值介于min和max之间的成员数量 |
| zrank | zrank keyname member 返回成员member在有序集合中的排名 |
| zscore | zscore keyname member 返回member的分值  |
| zrange | zrange keyname start stop [withscores] 返回有序集合中排名介于start和stop之间的members，如果给定了withscores，那么成员的分值也一并返回 |



```commandline
127.0.0.1:6379[3]> zadd myorderset 50 James
(integer) 1
127.0.0.1:6379[3]> zrange myorderset 0 -1
1) "James"
2) "Niko"
3) "Tom"
4) "Helen"
5) "Marry"
6) "test"
127.0.0.1:6379[3]> zrange myorderset 0 -1 withscores
 1) "James"
 2) "50"
 3) "Niko"
 4) "100"
 5) "Tom"
 6) "101"
 7) "Helen"
 8) "102"
 9) "Marry"
10) "103"
11) "test"
12) "1001"
127.0.0.1:6379[3]> zrem myorderset Tom
(integer) 1
127.0.0.1:6379[3]> zrange myorderset 0 -1 withscores
 1) "James"
 2) "50"
 3) "Niko"
 4) "100"
 5) "Helen"
 6) "102"
 7) "Marry"
 8) "103"
 9) "test"
10) "1001"
127.0.0.1:6379[3]> zcard myorderset
(integer) 5
127.0.0.1:6379[3]> zincrby myorderset 99 James
"149"
127.0.0.1:6379[3]> zrange myorderset 0 -1 withscores
 1) "Niko"
 2) "100"
 3) "Helen"
 4) "102"
 5) "Marry"
 6) "103"
 7) "James"
 8) "149"
 9) "test"
10) "1001"
127.0.0.1:6379[3]> zcount myorderset 100 200
(integer) 4
127.0.0.1:6379[3]> zrank myorderset test
(integer) 4
127.0.0.1:6379[3]> zscore myorderset test
"1001"
```

**有序集合的范围性命令**


| 命令 | 用例和描述 |
| --- | --- |
| zrevrank | zrevrank keyname member 降序排列，获取member的排名 |
| zrevrange | zrevrange keyname start stop [withscores] 降序排列，返回集合中给定范围的元素 |
| zrangebyscore | zrangebyscore keyname min max [withscores][limit offset count] 返回分值位于min~max之间的所有成员 |
| zrevrangebyscore | zrevrangebyscore keyname max min [withscores][limit offset count] 降序排列，返回分支位于max~min之间的所有成员 |
| zremrangebyrank | zremrangebyrank keyname start stop 移除集合中排名位于start~stop之间的元素 |
| zremrangebyscore | zremrangebyscore keyname min max 移除集合中分值位于min~max之间的元素 |
| zinterstore | zinterstore destkey keycount key [key…] [weights weight [weight…]] [aggregate sum|min|max] 对给定的有序集合执行交集运算,keycount为显式指定的集合个数 |
| zunionstore | zunionstore destkey keycount key [key…] [weights weight [weight…]] [aggregate sum|min|max] 对给定的有序集合执行并集运算 |



```commandline
127.0.0.1:6379[3]> zrange yourorderset 0 -1 withscores
1) "Tom"
2) "100"
3) "James"
4) "101"
127.0.0.1:6379[3]> zrange myorderset 0 -1 withscores
 1) "Niko"
 2) "100"
 3) "Helen"
 4) "102"
 5) "Marry"
 6) "103"
 7) "James"
 8) "149"
 9) "test"
10) "1001"
127.0.0.1:6379[3]> ZINTERSTORE interorderset 2  myorderset yourorderset
(integer) 1
127.0.0.1:6379[3]> zrange interorderset 0 -1 withscores
1) "James"
2) "250"
127.0.0.1:6379[3]> zunionstore unionorderstore 2 myorderset yourorderset
(integer) 6
127.0.0.1:6379[3]> zrange unionorderstore 0 -1 withscores
 1) "Niko"
 2) "100"
 3) "Tom"
 4) "100"
 5) "Helen"
 6) "102"
 7) "Marry"
 8) "103"
 9) "James"
10) "250"
11) "test"
12) "1001"
```

## 2.6 发布与订阅
略

## 2,7 其他命令

**排序**
sort可以根据字符串、列表、集合、有序集合、散列这5种键里面存储着的数据对其排序。类似于关系数据库中的order by语句。
| 命令 | 用例和描述 |
| --- | --- |
| sort | sort sourcekey [by pattern] [limit offset count] [get pattern [get pattern…]] [asc|desc] [alpha] [store dest-key]  根据给定的选项，对输入列表、集合或者有序集合进行排序，然后返回或者存储排序的结果 |


```commandline
127.0.0.1:6379[3]> rpush mysort 1 3 5 5 2 2 7 8 1 9 7
(integer) 12
127.0.0.1:6379[3]> SORT mysort
(error) ERR One or more scores can't be converted into double
127.0.0.1:6379[3]> lrange mysort 0 -1
 1) "1"
 2) "3"
 3) "5"
 4) "5"
 5) "2"
 6) "2"
 7) "7"
 8) "8"
 9) "1"
10) "9"
11) "7"
127.0.0.1:6379[3]> sort mysort
 1) "1"
 2) "1"
 3) "2"
 4) "2"
 5) "3"
 6) "5"
 7) "5"
 8) "7"
 9) "7"
10) "8"
11) "9"
127.0.0.1:6379[3]> sort mylist alpha
1) "NikoBelic"
2) "a"
127.0.0.1:6379[3]> sort mylist alpha desc
1) "a"
2) "NikoBelic"
```

## 2.8 键的过期时间


| 命令 | 示例和描述 |
| --- | --- |
| persist | persist keyname 移除键的过期时间 |
| ttl | ttl keyname 查看给定键距离过期还有多少秒 |
| expire | expire keyname seconds 让给定键在指定的秒数之后过期 |
| expireat | expireat keyname timestamp 将给定键的过期时间设置为给定的Unix时间戳 |
| pttl | pttl keyname 查看给定键距离过期时间还有多少毫秒，Redis2.6+ |
| pexpire | pexpire keyname milliseconds，让给定键在指定的毫秒数之后过期。Redis2.6+ |
| pexpireat | pexpireat keyname timestamp-milliseconds 将一个毫秒级精度的Unix时间戳设置为给定键的的过期时间，Redis2.6+ |



下一章：编写Java版的Redis操作工具类。

