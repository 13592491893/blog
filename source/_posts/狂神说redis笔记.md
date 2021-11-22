---
abbrlink: 54863
title: 狂神说redis笔记
comments: true
toc: true
description: 狂神说redis笔记
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - redis
tags:
  - java
  - redis
  - nosql
date: 2020-6-21 16:00:00
---
# 狂神说redis笔记（一）

# 一、Nosql概述

## 1、单机Mysql时代

90年代,一个网站的访问量一般不会太大，单个数据库完全够用。随着用户增多，网站出现以下问题：

1. 数据量增加到一定程度，单机数据库就放不下了
2. 数据的索引（B+ Tree）,一个机器内存也存放不下
3. 访问量变大后（读写混合），一台服务器承受不住。

[![img](https://img-blog.csdnimg.cn/2020082010365930.png#pic_center)](https://img-blog.csdnimg.cn/2020082010365930.png#pic_center)

## 2、Memcached(缓存) + Mysql + 垂直拆分（读写分离）

网站80%的情况都是在读，每次都要去查询数据库的话就十分的麻烦！所以说我们希望减轻数据库的压力，我们可以使用缓存来保证效率！

[![img](https://img-blog.csdnimg.cn/20200820103713734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200820103713734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)

优化过程经历了以下几个过程：

1. 优化数据库的数据结构和索引(难度大)
2. 文件缓存，通过IO流获取比每次都访问数据库效率略高，但是流量爆炸式增长时候，IO流也承受不了
3. MemCache,当时最热门的技术，通过在数据库和数据库访问层之间加上一层缓存，第一次访问时查询数据库，将结果保存到缓存，后续的查询先检查缓存，若有直接拿去使用，效率显著提升。

## 3、分库分表 + 水平拆分 + Mysql集群

[![img](https://img-blog.csdnimg.cn/20200820103739584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200820103739584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)

## 4、如今最近的年代

如今信息量井喷式增长，各种各样的数据出现（用户定位数据，图片数据等），大数据的背景下关系型数据库（RDBMS）无法满足大量数据要求。Nosql数据库就能轻松解决这些问题。目前一个基本的互联网项目：

[![img](https://img-blog.csdnimg.cn/20200820103804572.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200820103804572.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)

## 5、为什么要用NoSQL ？

用户的个人信息，社交网络，地理位置。用户自己产生的数据，用户日志等等爆发式增长！这时候我们就需要使用NoSQL数据库的，Nosql可以很好的处理以上的情况！

### 什么是Nosql

NoSQL = Not Only SQL（不仅仅是SQL）

Not Only Structured Query Language

关系型数据库：列+行，同一个表下数据的结构是一样的。

非关系型数据库：数据存储没有固定的格式，并且可以进行横向扩展。

NoSQL泛指非关系型数据库，随着web2.0互联网的诞生，传统的关系型数据库很难对付web2.0时代！尤其是超大规模的高并发的社区，暴露出来很多难以克服的问题，NoSQL在当今大数据环境下发展的十分迅速，Redis是发展最快的。

### Nosql特点

1.方便扩展（数据之间没有关系，很好扩展！）

2.大数据量高性能（Redis一秒可以写8万次，读11万次，NoSQL的缓存记录级，是一种细粒度的缓存，性能会比较高！）

3.数据类型是多样型的！（不需要事先设计数据库，随取随用）

4.传统的 RDBMS 和 NoSQL

> 传统的 RDBMS(关系型数据库)
>
> > 结构化组织
> > SQL
> > 数据和关系都存在单独的表中 row col
> > 操作，数据定义语言
> > 严格的一致性
> > 基础的事务
> > ...
>
> Nosql
>
> > 不仅仅是数据
> > 没有固定的查询语言
> > 键值对存储，列存储，文档存储，图形数据库（社交关系）
> > 最终一致性
> > CAP定理和BASE
> > 高性能，高可用，高扩展
> > ...

5.大数据时代的3V ：主要是描述问题的

海量Velume

多样Variety

实时Velocity

6.大数据时代的3高 ： 主要是对程序的要求

高并发

高可扩

高性能

真正在公司中的实践：NoSQL + RDBMS 一起使用才是最强的。

# 二、Redis入门

## Redis是什么？

Redis（Remote Dictionary Server )，即远程字典服务。

是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

## Redis能干什么？

1. 内存存储、持久化，内存是断电即失的，所以需要持久化（RDB、AOF）
2. 高效率、用于高速缓冲
3. 发布订阅系统
4. 地图信息分析
5. 计时器、计数器(eg：浏览量)
6. 。。。

## 特性

1. 多样的数据类型
2. 持久化
3. 集群
4. 事务
5. ...

## 环境搭建（略）

## 性能测试

**redis-benchmark：**Redis官方提供的性能测试工具，参数选项如下：

[![img](https://img-blog.csdnimg.cn/20200513214125892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513214125892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

简单测试：



```
# 测试：100个并发连接 100000请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

结果：

[![img](https://img-blog.csdnimg.cn/20200820104343472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200820104343472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)

## 基础知识

redis默认有16个数据库

[![img](https://img-blog.csdnimg.cn/20200820104357466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200820104357466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)

默认使用的第0个;

16个数据库为：DB 0~DB 15 默认使用DB 0 ，可以使用`select n`切换到DB n，`dbsize`可以查看当前数据库的大小，与key数量相关。



```
127.0.0.1:6379> config get databases # 命令行查看数据库数量databases
1) "databases"
2) "16"
127.0.0.1:6379> select 8 # 切换数据库 DB 8
OK
127.0.0.1:6379[8]> dbsize # 查看数据库大小
(integer) 0
不同数据库之间 数据是不能互通的，并且dbsize 是根据库中key的个数。
127.0.0.1:6379> set name sakura
OK
127.0.0.1:6379> SELECT 8
OK
127.0.0.1:6379[8]> get name # db8中并不能获取db0中的键值对。
(nil)
127.0.0.1:6379[8]> DBSIZE
(integer) 0
127.0.0.1:6379[8]> SELECT 0
OK
127.0.0.1:6379> keys *

"counter:rand_int"
"mylist"
"name"
"key:rand_int"
"myset:rand_int"
127.0.0.1:6379> DBSIZE # size和key个数相关
(integer) 5
```

`keys *` ：查看当前数据库中所有的key。

`flushdb`：清空当前数据库中的键值对。

`flushall`：清空所有数据库的键值对。

> Redis是单线程的，Redis是基于内存操作的。

所以Redis的性能瓶颈不是CPU,而是机器内存和网络带宽。

那么为什么Redis的速度如此快呢，性能这么高呢？QPS达到10W+

> Redis为什么单线程还这么快？

- 误区1：高性能的服务器一定是多线程的？
- 误区2：多线程（CPU上下文会切换！）一定比单线程效率高！

核心：Redis是将所有的数据放在内存中的，所以说使用单线程去操作效率就是最高的，多线程（CPU上下文会切换：耗时的操作！），对于内存系统来说，如果没有上下文切换效率就是最高的，多次读写都是在一个CPU上的，在内存存储数据情况下，单线程就是最佳的方案。

# 三、五大数据类型

 Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持字符串、哈希表、列表、集合、有序集合，位图，hyperloglogs等数据类型。内置复制、Lua脚本、LRU收回、事务以及不同级别磁盘持久化功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动分区。

## Redis-key

在redis中无论什么数据类型，在数据库中都是以key-value形式保存，通过进行对Redis-key的操作，来完成对数据库中数据的操作。

下面学习的命令：

- `exists key`：判断键是否存在
- `del key`：删除键值对
- `move key db`：将键值对移动到指定数据库
- `expire key second`：设置键值对的过期时间
- `type key`：查看value的数据类型

　　　



```
127.0.0.1:6379> keys * # 查看当前数据库所有key
(empty list or set)
127.0.0.1:6379> set name qinjiang # set key
OK
127.0.0.1:6379> set age 20
OK
127.0.0.1:6379> keys *
1) "age"
2) "name"
127.0.0.1:6379> move age 1 # 将键值对移动到指定数据库
(integer) 1
127.0.0.1:6379> EXISTS age # 判断键是否存在
(integer) 0 # 不存在
127.0.0.1:6379> EXISTS name
(integer) 1 # 存在
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> keys *
1) "age"
127.0.0.1:6379[1]> del age # 删除键值对
(integer) 1 # 删除个数
127.0.0.1:6379> set age 20
OK
127.0.0.1:6379> EXPIRE age 15 # 设置键值对的过期时间
(integer) 1 # 设置成功 开始计数
127.0.0.1:6379> ttl age # 查看key的过期剩余时间
(integer) 13
127.0.0.1:6379> ttl age
(integer) 11
127.0.0.1:6379> ttl age
(integer) 9
127.0.0.1:6379> ttl age
(integer) -2 # -2 表示key过期，-1表示key未设置过期时间
127.0.0.1:6379> get age # 过期的key 会被自动delete
(nil)
127.0.0.1:6379> keys *

"name"

127.0.0.1:6379> type name # 查看value的数据类型
string
```



关于TTL命令

Redis的key，通过TTL命令返回key的过期时间，一般来说有3种：

1. 当前key没有设置过期时间，所以会返回-1.
2. 当前key有设置过期时间，而且key已经过期，所以会返回-2.
3. 当前key有设置过期时间，且key还没有过期，故会返回key的正常剩余时间.

关于重命名`RENAME`和`RENAMENX`

1. `RENAME key newkey`修改 key 的名称
2. `RENAMENX key newkey`仅当 newkey 不存在时，将 key 改名为 newkey 。

## String(字符串)

普通的set、get直接略过。

常用命令及其示例：

`APPEND key value`: 向指定的key的value后追加字符串



```
127.0.0.1:6379> set msg hello 
OK 
127.0.0.1:6379> append msg " world" 
(integer) 11 
127.0.0.1:6379> get msg 
“hello world”
```

`DECR/INCR key`: 将指定key的value数值进行+1/-1(仅对于数字)



```
127.0.0.1:6379> set age 20 
OK 
127.0.0.1:6379> incr age 
(integer) 21 
127.0.0.1:6379> decr age 
(integer) 20
```

`INCRBY/DECRBY key n`: 按指定的步长对数值进行加减



```
127.0.0.1:6379> INCRBY age 5
(integer) 25 
127.0.0.1:6379> DECRBY age 10 
(integer) 15
```

`INCRBYFLOAT key n`: 为数值加上浮点型数值



```
127.0.0.1:6379> INCRBYFLOAT age 5.2 
“20.2”
```

`STRLEN key`: 获取key保存值的字符串长度



```
127.0.0.1:6379> get msg 
“hello world” 
127.0.0.1:6379> STRLEN msg 
(integer) 11
```

`GETRANGE key start end`: 按起止位置获取字符串（闭区间，起止位置都取）



```
127.0.0.1:6379> get msg 
“hello world” 
127.0.0.1:6379> GETRANGE msg 3 9 
“lo worl”
```

`SETRANGE key offset value`:用指定的value 替换key中 offset开始的值



```
127.0.0.1:6379> set msg hello
OK
127.0.0.1:6379> setrange msg 2 hello
(integer) 7
127.0.0.1:6379> get msg
"hehello"
127.0.0.1:6379> set msg2 world
OK
127.0.0.1:6379> setrange msg2 2 ww
(integer) 5
127.0.0.1:6379> get msg2
"wowwd"
```

`GETSET key value`: 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。



```
127.0.0.1:6379> GETSET msg test 
“hello world”
```

`SETNX key value`: 仅当key不存在时进行set



```
127.0.0.1:6379> SETNX msg test 
(integer) 0 
127.0.0.1:6379> SETNX name sakura 
(integer) 1
```

`SETEX key seconds value`: set 键值对并设置过期时间



```
127.0.0.1:6379> setex name 10 root 
OK 
127.0.0.1:6379> get name 
(nil)
```

`MSET key1 value1 [key2 value2..]`: 批量set键值对



```
127.0.0.1:6379> MSET k1 v1 k2 v2 k3 v3 
OK
```

`MSETNX key1 value1 [key2 value2..]`: 批量设置键值对，仅当参数中所有的key都不存在时执行



```
127.0.0.1:6379> MSETNX k1 v1 k4 v4 
(integer) 0
```

`MGET key1 [key2..]`: 批量获取多个key保存的值



```
127.0.0.1:6379> MGET k1 k2 k3 
1) “v1” 
2) “v2” 
3) “v3”
```

`PSETEX key milliseconds value`: 和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间

String类似的使用场景：value除了是字符串还可以是数字，用途举例：

- 计数器
- 统计多单位的数量：uid:123666：follow 0
- 粉丝数
- 对象存储缓存

## List(列表)

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

首先我们列表，可以经过规则定义将其变为队列、栈、双端队列等。

[![img](https://img-blog.csdnimg.cn/20200820104440398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200820104440398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RERERlbmdf,size_16,color_FFFFFF,t_70#pic_center)

正如图Redis中List是可以进行双端操作的，所以命令也就分为了LXXX和RLLL两类，有时候L也表示List例如LLEN

- `LPUSH/RPUSH key value1[value2..]`从左边/右边向列表中PUSH值(一个或者多个)。
- `LRANGE key start end` 获取list 起止元素==（索引从左往右 递增）==
- `LPUSHX/RPUSHX key value` 向已存在的列名中push值（一个或者多个）
- `LINSERT key BEFORE|AFTER pivot value` 在指定列表元素的前/后 插入value
- `LLEN key` 查看列表长度
- `LINDEX key index` 通过索引获取列表元素
- `LSET key index value` 通过索引为元素设值
- `LPOP/RPOP key` 从最左边/最右边移除值 并返回
- `RPOPLPUSH source destination` 将列表的尾部(右)最后一个值弹出，并返回，然后加到另一个列表的头部
- `LTRIM key start end` 通过下标截取指定范围内的列表
- `LREM key count value` List中是允许value重复的 count > 0：从头部开始搜索 然后删除指定的value 至多删除count个 count < 0：从尾部开始搜索… count = 0：删除列表中所有的指定value。
- `BLPOP/BRPOP key1[key2] timout` 移出并获取列表的第一个/最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
- `BRPOPLPUSH source destination timeout` 和RPOPLPUSH功能相同，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

代码示例：



```
---------------------------LPUSH---RPUSH---LRANGE--------------------------------
127.0.0.1:6379> LPUSH mylist k1 # LPUSH mylist=>{1}
(integer) 1
127.0.0.1:6379> LPUSH mylist k2 # LPUSH mylist=>{2,1}
(integer) 2
127.0.0.1:6379> RPUSH mylist k3 # RPUSH mylist=>{2,1,3}
(integer) 3
127.0.0.1:6379> get mylist # 普通的get是无法获取list值的
(error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> LRANGE mylist 0 4 # LRANGE 获取起止位置范围内的元素

"k2"
"k1"
"k3"
127.0.0.1:6379> LRANGE mylist 0 2
"k2"
"k1"
"k3"
127.0.0.1:6379> LRANGE mylist 0 1
"k2"
"k1"
127.0.0.1:6379> LRANGE mylist 0 -1 # 获取全部元素
"k2"
"k1"
"k3"

---------------------------LPUSHX---RPUSHX-----------------------------------
127.0.0.1:6379> LPUSHX list v1 # list不存在 LPUSHX失败
(integer) 0
127.0.0.1:6379> LPUSHX list v1 v2
(integer) 0
127.0.0.1:6379> LPUSHX mylist k4 k5 # 向mylist中 左边 PUSH k4 k5
(integer) 5
127.0.0.1:6379> LRANGE mylist 0 -1

"k5"
"k4"
"k2"
"k1"
"k3"

---------------------------LINSERT--LLEN--LINDEX--LSET----------------------------
127.0.0.1:6379> LINSERT mylist after k2 ins_key1 # 在k2元素后 插入ins_key1
(integer) 6
127.0.0.1:6379> LRANGE mylist 0 -1

"k5"
"k4"
"k2"
"ins_key1"
"k1"
"k3"
127.0.0.1:6379> LLEN mylist # 查看mylist的长度
(integer) 6
127.0.0.1:6379> LINDEX mylist 3 # 获取下标为3的元素
"ins_key1"
127.0.0.1:6379> LINDEX mylist 0
"k5"
127.0.0.1:6379> LSET mylist 3 k6 # 将下标3的元素 set值为k6
OK
127.0.0.1:6379> LRANGE mylist 0 -1
"k5"
"k4"
"k2"
"k6"
"k1"
"k3"

---------------------------LPOP--RPOP--------------------------
127.0.0.1:6379> LPOP mylist # 左侧(头部)弹出
"k5"
127.0.0.1:6379> RPOP mylist # 右侧(尾部)弹出
"k3"
---------------------------RPOPLPUSH--------------------------
127.0.0.1:6379> LRANGE mylist 0 -1

"k4"
"k2"
"k6"
"k1"
127.0.0.1:6379> RPOPLPUSH mylist newlist # 将mylist的最后一个值(k1)弹出，加入到newlist的头部
"k1"
127.0.0.1:6379> LRANGE newlist 0 -1
"k1"
127.0.0.1:6379> LRANGE mylist 0 -1
"k4"
"k2"
"k6"

---------------------------LTRIM--------------------------
127.0.0.1:6379> LTRIM mylist 0 1 # 截取mylist中的 0~1部分
OK
127.0.0.1:6379> LRANGE mylist 0 -1

"k4"
"k2"

初始 mylist: k2,k2,k2,k2,k2,k2,k4,k2,k2,k2,k2
---------------------------LREM--------------------------
127.0.0.1:6379> LREM mylist 3 k2 # 从头部开始搜索 至多删除3个 k2
(integer) 3
删除后：mylist: k2,k2,k2,k4,k2,k2,k2,k2
127.0.0.1:6379> LREM mylist -2 k2 #从尾部开始搜索 至多删除2个 k2
(integer) 2
删除后：mylist: k2,k2,k2,k4,k2,k2
---------------------------BLPOP--BRPOP--------------------------
mylist: k2,k2,k2,k4,k2,k2
newlist: k1
127.0.0.1:6379> BLPOP newlist mylist 30 # 从newlist中弹出第一个值，mylist作为候选

"newlist" # 弹出
"k1"
127.0.0.1:6379> BLPOP newlist mylist 30
"mylist" # 由于newlist空了 从mylist中弹出
"k2"
127.0.0.1:6379> BLPOP newlist 30
(30.10s) # 超时了

127.0.0.1:6379> BLPOP newlist 30 # 我们连接另一个客户端向newlist中push了test, 阻塞被解决。

"newlist"
"test"
(12.54s)
```

### 小结

- list实际上是一个链表，before Node after , left, right 都可以插入值
- 如果key不存在，则创建新的链表
- 如果key存在，新增内容
- 如果移除了所有值，空链表，也代表不存在
- 在两边插入或者改动值，效率最高！修改中间元素，效率相对较低

应用：

消息排队！消息队列（Lpush Rpop）,栈（Lpush Lpop）

## Set(集合）

Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

- `SADD key member1[member2..]` 向集合中无序增加一个/多个成员
- `SCARD key` 获取集合的成员数
- `SMEMBERS key` 返回集合中所有的成员
- `SISMEMBER key member` 查询member元素是否是集合的成员,结果是无序的
- `SRANDMEMBER key [count]` 随机返回集合中count个成员，count缺省值为1
- `SPOP key [count]` 随机移除并返回集合中count个成员，count缺省值为1
- `SMOVE source destination member` 将source集合的成员member移动到destination集合
- `SREM key member1[member2..]` 移除集合中一个/多个成员
- `SDIFF key1[key2..]` 返回所有集合的差集 key1- key2 - …
- `SDIFFSTORE destination key1[key2..]` 在SDIFF的基础上，将结果保存到集合中==(覆盖)==。不能保存到其他类型key噢！
- `SINTER key1 [key2..]` 返回所有集合的交集
- `SINTERSTORE destination key1[key2..]` 在SINTER的基础上，存储结果到集合中。覆盖
- `SUNION key1 [key2..]` 返回所有集合的并集
- `SUNIONSTORE destination key1 [key2..]` 在SUNION的基础上，存储结果到及和张。覆盖
- `SSCAN KEY [MATCH pattern] [COUNT count]` 在大量数据环境下，使用此命令遍历集合中元素，每次遍历部分

代码示例



```
---------------SADD--SCARD--SMEMBERS--SISMEMBER--------------------
127.0.0.1:6379> SADD myset m1 m2 m3 m4 # 向myset中增加成员 m1~m4
(integer) 4
127.0.0.1:6379> SCARD myset # 获取集合的成员数目
(integer) 4
127.0.0.1:6379> smembers myset # 获取集合中所有成员

"m4"
"m3"
"m2"
"m1"
127.0.0.1:6379> SISMEMBER myset m5 # 查询m5是否是myset的成员
(integer) 0 # 不是，返回0
127.0.0.1:6379> SISMEMBER myset m2
(integer) 1 # 是，返回1
127.0.0.1:6379> SISMEMBER myset m3
(integer) 1

---------------------SRANDMEMBER--SPOP----------------------------------
127.0.0.1:6379> SRANDMEMBER myset 3 # 随机返回3个成员

"m2"
"m3"
"m4"
127.0.0.1:6379> SRANDMEMBER myset # 随机返回1个成员
"m3"
127.0.0.1:6379> SPOP myset 2 # 随机移除并返回2个成员
"m1"
"m4"

将set还原到{m1,m2,m3,m4}
---------------------SMOVE--SREM----------------------------------------
127.0.0.1:6379> SMOVE myset newset m3 # 将myset中m3成员移动到newset集合
(integer) 1
127.0.0.1:6379> SMEMBERS myset

"m4"
"m2"
"m1"
127.0.0.1:6379> SMEMBERS newset
"m3"
127.0.0.1:6379> SREM newset m3 # 从newset中移除m3元素
(integer) 1
127.0.0.1:6379> SMEMBERS newset
(empty list or set)

下面开始是多集合操作,多集合操作中若只有一个参数默认和自身进行运算
setx=>{m1,m2,m4,m6}, sety=>{m2,m5,m6}, setz=>{m1,m3,m6}
-----------------------------SDIFF------------------------------------
127.0.0.1:6379> SDIFF setx sety setz # 等价于setx-sety-setz

"m4"
127.0.0.1:6379> SDIFF setx sety # setx - sety
"m4"
"m1"
127.0.0.1:6379> SDIFF sety setx # sety - setx
"m5"

-------------------------SINTER---------------------------------------
共同关注（交集）
127.0.0.1:6379> SINTER setx sety setz # 求 setx、sety、setx的交集

"m6"
127.0.0.1:6379> SINTER setx sety # 求setx sety的交集
"m2"
"m6"

-------------------------SUNION---------------------------------------
127.0.0.1:6379> SUNION setx sety setz # setx sety setz的并集

"m4"
"m6"
"m3"
"m2"
"m1"
"m5"
127.0.0.1:6379> SUNION setx sety # setx sety 并集
"m4"
"m6"
"m2"
"m1"
"m5"
```

## Hash（哈希）

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

Set就是一种简化的Hash,只变动key,而value使用默认值填充。可以将一个Hash表作为一个对象进行存储，表中存放对象的信息。

- `HSET key field value` 将哈希表 key 中的字段 field 的值设为 value 。重复设置同一个field会覆盖,返回0
- `HMSET key field1 value1 [field2 value2..]` 同时将多个 field-value (域-值)对设置到哈希表 key 中。
- `HSETNX key field value` 只有在字段 field 不存在时，设置哈希表字段的值。
- `HEXISTS key field` 查看哈希表 key 中，指定的字段是否存在。
- `HGET key field value` 获取存储在哈希表中指定字段的值
- `HMGET key field1 [field2..]` 获取所有给定字段的值
- `HGETALL key` 获取在哈希表key 的所有字段和值
- `HKEYS key` 获取哈希表key中所有的字段
- `HLEN key` 获取哈希表中字段的数量
- `HVALS key` 获取哈希表中所有值
- `HDEL key field1 [field2..]` 删除哈希表key中一个/多个field字段
- `HINCRBY key field n` 为哈希表 key 中的指定字段的整数值加上增量n，并返回增量后结果 一样只适用于整数型字段
- `HINCRBYFLOAT key field n` 为哈希表 key 中的指定字段的浮点数值加上增量 n。
- `HSCAN key cursor [MATCH pattern] [COUNT count]` 迭代哈希表中的键值对。

代码示例



```
------------------------HSET--HMSET--HSETNX----------------
127.0.0.1:6379> HSET studentx name sakura # 将studentx哈希表作为一个对象，设置name为sakura
(integer) 1
127.0.0.1:6379> HSET studentx name gyc # 重复设置field进行覆盖，并返回0
(integer) 0
127.0.0.1:6379> HSET studentx age 20 # 设置studentx的age为20
(integer) 1
127.0.0.1:6379> HMSET studentx sex 1 tel 15623667886 # 设置sex为1，tel为15623667886
OK
127.0.0.1:6379> HSETNX studentx name gyc # HSETNX 设置已存在的field
(integer) 0 # 失败
127.0.0.1:6379> HSETNX studentx email 12345@qq.com
(integer) 1 # 成功
----------------------HEXISTS--------------------------------
127.0.0.1:6379> HEXISTS studentx name # name字段在studentx中是否存在
(integer) 1 # 存在
127.0.0.1:6379> HEXISTS studentx addr
(integer) 0 # 不存在
-------------------HGET--HMGET--HGETALL-----------
127.0.0.1:6379> HGET studentx name # 获取studentx中name字段的value
"gyc"
127.0.0.1:6379> HMGET studentx name age tel # 获取studentx中name、age、tel字段的value

"gyc"
"20"
"15623667886"
127.0.0.1:6379> HGETALL studentx # 获取studentx中所有的field及其value
"name"
"gyc"
"age"
"20"
"sex"
"1"
"tel"
"15623667886"
"email"
"12345@qq.com"

--------------------HKEYS--HLEN--HVALS--------------
127.0.0.1:6379> HKEYS studentx # 查看studentx中所有的field

"name"
"age"
"sex"
"tel"
"email"
127.0.0.1:6379> HLEN studentx # 查看studentx中的字段数量
(integer) 5
127.0.0.1:6379> HVALS studentx # 查看studentx中所有的value
"gyc"
"20"
"1"
"15623667886"
"12345@qq.com"

-------------------------HDEL--------------------------
127.0.0.1:6379> HDEL studentx sex tel # 删除studentx 中的sex、tel字段
(integer) 2
127.0.0.1:6379> HKEYS studentx

"name"
"age"
"email"

-------------HINCRBY--HINCRBYFLOAT------------------------
127.0.0.1:6379> HINCRBY studentx age 1 # studentx的age字段数值+1
(integer) 21
127.0.0.1:6379> HINCRBY studentx name 1 # 非整数字型字段不可用
(error) ERR hash value is not an integer
127.0.0.1:6379> HINCRBYFLOAT studentx weight 0.6 # weight字段增加0.6
"90.8"
```



 Hash变更的数据user name age，尤其是用户信息之类的，经常变动的信息！Hash更适合于对象的存储，Sring更加适合字符串存储！

## Zset（有序集合）

不同的是每个元素都会关联一个double类型的分数（score）。redis正是通过分数来为集合中的成员进行从小到大的排序。

score相同：按字典顺序排序

有序集合的成员是唯一的,但分数(score)却可以重复。

- `ZADD key score member1 [score2 member2]` 向有序集合添加一个或多个成员，或者更新已存在成员的分数
- `ZCARD key` 获取有序集合的成员数
- `ZCOUNT key min max` 计算在有序集合中指定区间score的成员数
- `ZINCRBY key n member` 有序集合中对指定成员的分数加上增量 n
- `ZSCORE key member` 返回有序集中，成员的分数值
- `ZRANK key member` 返回有序集合中指定成员的索引
- `ZRANGE key start end` 通过索引区间返回有序集合成指定区间内的成员
- `ZRANGEBYLEX key min max` 通过字典区间返回有序集合的成员
- `ZRANGEBYSCORE key min max` 通过分数返回有序集合指定区间内的成员==-inf 和 +inf分别表示最小最大值，只支持开区间()==
- `ZLEXCOUNT key min max` 在有序集合中计算指定字典区间内成员数量
- `ZREM key member1 [member2..]` 移除有序集合中一个/多个成员
- `ZREMRANGEBYLEX key min max` 移除有序集合中给定的字典区间的所有成员
- `ZREMRANGEBYRANK key start stop` 移除有序集合中给定的排名区间的所有成员
- `ZREMRANGEBYSCORE key min max` 移除有序集合中给定的分数区间的所有成员
- `ZREVRANGE key start end` 返回有序集中指定区间内的成员，通过索引，分数从高到底
- `ZREVRANGEBYSCORRE key max min` 返回有序集中指定分数区间内的成员，分数从高到低排序
- `ZREVRANGEBYLEX key max min` 返回有序集中指定字典区间内的成员，按字典顺序倒序
- `ZREVRANK key member` 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
- `ZINTERSTORE destination numkeys key1 [key2 ..]` 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中，numkeys：表示参与运算的集合数，将score相加作为结果的score
- `ZUNIONSTORE destination numkeys key1 [key2..]` 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
- `ZSCAN key cursor [MATCH pattern\] [COUNT count]` 迭代有序集合中的元素（包括元素成员和元素分值）

代码示例



```
-------------------ZADD--ZCARD--ZCOUNT--------------
127.0.0.1:6379> ZADD myzset 1 m1 2 m2 3 m3 # 向有序集合myzset中添加成员m1 score=1 以及成员m2 score=2..
(integer) 2
127.0.0.1:6379> ZCARD myzset # 获取有序集合的成员数
(integer) 2
127.0.0.1:6379> ZCOUNT myzset 0 1 # 获取score在 [0,1]区间的成员数量
(integer) 1
127.0.0.1:6379> ZCOUNT myzset 0 2
(integer) 2
----------------ZINCRBY--ZSCORE--------------------------
127.0.0.1:6379> ZINCRBY myzset 5 m2 # 将成员m2的score +5
"7"
127.0.0.1:6379> ZSCORE myzset m1 # 获取成员m1的score
"1"
127.0.0.1:6379> ZSCORE myzset m2
"7"
--------------ZRANK--ZRANGE-----------------------------------
127.0.0.1:6379> ZRANK myzset m1 # 获取成员m1的索引，索引按照score排序，score相同索引值按字典顺序顺序增加
(integer) 0
127.0.0.1:6379> ZRANK myzset m2
(integer) 2
127.0.0.1:6379> ZRANGE myzset 0 1 # 获取索引在 0~1的成员

"m1"
"m3"
127.0.0.1:6379> ZRANGE myzset 0 -1 # 获取全部成员
"m1"
"m3"
"m2"

testset=>{abc,add,amaze,apple,back,java,redis} score均为0
------------------ZRANGEBYLEX---------------------------------
127.0.0.1:6379> ZRANGEBYLEX testset - + # 返回所有成员

"abc"
"add"
"amaze"
"apple"
"back"
"java"
"redis"
127.0.0.1:6379> ZRANGEBYLEX testset - + LIMIT 0 3 # 分页 按索引显示查询结果的 0,1,2条记录
"abc"
"add"
"amaze"
127.0.0.1:6379> ZRANGEBYLEX testset - + LIMIT 3 3 # 显示 3,4,5条记录
"apple"
"back"
"java"
127.0.0.1:6379> ZRANGEBYLEX testset (- [apple # 显示 (-,apple] 区间内的成员
"abc"
"add"
"amaze"
"apple"
127.0.0.1:6379> ZRANGEBYLEX testset [apple [java # 显示 [apple,java]字典区间的成员
"apple"
"back"
"java"

-----------------------ZRANGEBYSCORE---------------------
127.0.0.1:6379> ZRANGEBYSCORE myzset 1 10 # 返回score在 [1,10]之间的的成员

"m1"
"m3"
"m2"
127.0.0.1:6379> ZRANGEBYSCORE myzset 1 5
"m1"
"m3"

--------------------ZLEXCOUNT-----------------------------
127.0.0.1:6379> ZLEXCOUNT testset - +
(integer) 7
127.0.0.1:6379> ZLEXCOUNT testset [apple [java
(integer) 3
------------------ZREM--ZREMRANGEBYLEX--ZREMRANGBYRANK--ZREMRANGEBYSCORE--------------------------------
127.0.0.1:6379> ZREM testset abc # 移除成员abc
(integer) 1
127.0.0.1:6379> ZREMRANGEBYLEX testset [apple [java # 移除字典区间[apple,java]中的所有成员
(integer) 3
127.0.0.1:6379> ZREMRANGEBYRANK testset 0 1 # 移除排名0~1的所有成员
(integer) 2
127.0.0.1:6379> ZREMRANGEBYSCORE myzset 0 3 # 移除score在 [0,3]的成员
(integer) 2
testset=> {abc,add,apple,amaze,back,java,redis} score均为0
myzset=> {(m1,1),(m2,2),(m3,3),(m4,4),(m7,7),(m9,9)}
----------------ZREVRANGE--ZREVRANGEBYSCORE--ZREVRANGEBYLEX-----------
127.0.0.1:6379> ZREVRANGE myzset 0 3 # 按score递减排序，然后按索引，返回结果的 0~3

"m9"
"m7"
"m4"
"m3"
127.0.0.1:6379> ZREVRANGE myzset 2 4 # 返回排序结果的 索引的2~4
"m4"
"m3"
"m2"
127.0.0.1:6379> ZREVRANGEBYSCORE myzset 6 2 # 按score递减顺序 返回集合中分数在[2,6]之间的成员
"m4"
"m3"
"m2"
127.0.0.1:6379> ZREVRANGEBYLEX testset [java (add # 按字典倒序 返回集合中(add,java]字典区间的成员
"java"
"back"
"apple"
"amaze"

-------------------------ZREVRANK------------------------------
127.0.0.1:6379> ZREVRANK myzset m7 # 按score递减顺序，返回成员m7索引
(integer) 1
127.0.0.1:6379> ZREVRANK myzset m2
(integer) 4
mathscore=>{(xm,90),(xh,95),(xg,87)} 小明、小红、小刚的数学成绩
enscore=>{(xm,70),(xh,93),(xg,90)} 小明、小红、小刚的英语成绩
-------------------ZINTERSTORE--ZUNIONSTORE-----------------------------------
127.0.0.1:6379> ZINTERSTORE sumscore 2 mathscore enscore # 将mathscore enscore进行合并 结果存放到sumscore
(integer) 3
127.0.0.1:6379> ZRANGE sumscore 0 -1 withscores # 合并后的score是之前集合中所有score的和

"xm"
"160"
"xg"
"177"
"xh"
"188"

127.0.0.1:6379> ZUNIONSTORE lowestscore 2 mathscore enscore AGGREGATE MIN # 取两个集合的成员score最小值作为结果的
(integer) 3
127.0.0.1:6379> ZRANGE lowestscore 0 -1 withscores

"xm"
"70"
"xg"
"87"
"xh"
"93"
```

应用案例：

1. set排序 存储班级成绩表 工资表排序！
2. 普通消息，1.重要消息 2.带权重进行判断
3. 排行榜应用实现，取Top N测试

# 四、三种特殊数据类型

## Geospatial(地理位置)

使用经纬度定位地理坐标并用一个有序集合zset保存，所以zset命令也可以使用

- `geoadd key longitud(经度) latitude(纬度) member [..]` 将具体经纬度的坐标存入一个有序集合
- `geopos key member [member..]` 获取集合中的一个/多个成员坐标
- `geodist key member1 member2 [unit]` 返回两个给定位置之间的距离。默认以米作为单位。
- `georadius key longitude latitude radius m|km|mi|ft [WITHCOORD][WITHDIST] [WITHHASH] [COUNT count]` 以给定的经纬度为中心， 返回集合包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。
- `GEORADIUSBYMEMBER key member radius...` 功能与GEORADIUS相同，只是中心位置不是具体的经纬度，而是使用结合中已有的成员作为中心点。
- `geohash key member1 [member2..]` 返回一个或多个位置元素的Geohash表示。使用Geohash位置52点整数编码。

### 有效经纬度

- 有效的经度从-180度到180度。
- 有效的纬度从-85.05112878度到85.05112878度。

指定单位的参数 unit 必须是以下单位的其中一个：

m 表示单位为米。

km 表示单位为千米。

mi 表示单位为英里。

ft 表示单位为英尺。

### 关于GEORADIUS的参数

通过georadius就可以完成 附近的人功能

withcoord:带上坐标

withdist:带上距离，单位与半径单位相同

COUNT n : 只显示前n个(按距离递增排序)



```
----------------georadius---------------------
127.0.0.1:6379> GEORADIUS china:city 120 30 500 km withcoord withdist # 查询经纬度(120,30)坐标500km半径内的成员
1) 1) "hangzhou"
   2) "29.4151"
   3) 1) "120.20000249147415"
      2) "30.199999888333501"
2) 1) "shanghai"
   2) "205.3611"
   3) 1) "121.40000134706497"
      2) "31.400000253193539"
------------geohash---------------------------
127.0.0.1:6379> geohash china:city yichang shanghai # 获取成员经纬坐标的geohash表示

"wmrjwbr5250"
"wtw6ds0y300"
```

## Hyperloglog(基数统计)

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。

花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基数。

因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

其底层使用string数据类型。

### 什么是基数？

数据集中不重复的元素的个数。

### 应用场景：

网页的访问量（UV）：一个用户多次访问，也只能算作一个人。

传统实现，存储用户的id,然后每次进行比较。当用户变多之后这种方式及其浪费空间，而我们的目的只是计数，Hyperloglog就能帮助我们利用最小的空间完成。

- `PFADD key element1 [elememt2..]` 添加指定元素到 HyperLogLog中
- `PFCOUNT key [key]` 返回给定 HyperLogLog 的基数估算值。
- `PFMERGE destkey sourcekey [sourcekey..]` 将多个 HyperLogLog 合并为一个 HyperLogLog

代码示例



```
----------PFADD--PFCOUNT---------------------
127.0.0.1:6379> PFADD myelemx a b c d e f g h i j k # 添加元素
(integer) 1
127.0.0.1:6379> type myelemx # hyperloglog底层使用String
string
127.0.0.1:6379> PFCOUNT myelemx # 估算myelemx的基数
(integer) 11
127.0.0.1:6379> PFADD myelemy i j k z m c b v p q s
(integer) 1
127.0.0.1:6379> PFCOUNT myelemy
(integer) 11
----------------PFMERGE-----------------------
127.0.0.1:6379> PFMERGE myelemz myelemx myelemy # 合并myelemx和myelemy 成为myelemz
OK
127.0.0.1:6379> PFCOUNT myelemz # 估算基数
(integer) 17
```



## BitMaps(位图)

使用位存储，信息状态只有 0 和 1

Bitmap是一串连续的2进制数字（0或1），每一位所在的位置为偏移(offset)，在bitmap上可执行AND,OR,XOR,NOT以及其它位操作。

应用场景: 签到统计、状态统计

- `setbit key offset value` 为指定key的offset位设置值
- `getbit key offset` 获取offset位的值
- `bitcount key [start end]` 统计字符串被设置为1的bit数，也可以指定统计范围按字节
- `bitop operration destkey key[key..]` 对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。
- `BITPOS key bit [start] [end]` 返回字符串里面第一个被设置为1或者0的bit位。start和end只能按字节,不能按位

代码示例



```
------------setbit--getbit--------------
127.0.0.1:6379> setbit sign 0 1 # 设置sign的第0位为 1 
(integer) 0
127.0.0.1:6379> setbit sign 2 1 # 设置sign的第2位为 1  不设置默认 是0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 5 1
(integer) 0
127.0.0.1:6379> type sign
string
127.0.0.1:6379> getbit sign 2 # 获取第2位的数值
(integer) 1
127.0.0.1:6379> getbit sign 3
(integer) 1
127.0.0.1:6379> getbit sign 4 # 未设置默认是0
(integer) 0
-----------bitcount----------------------------
127.0.0.1:6379> BITCOUNT sign # 统计sign中为1的位数
(integer) 4
```



# 五、事务

Redis的单条命令是保证原子性的，但是redis事务不能保证原子性



```
Redis事务本质：一组命令的集合。
----------------- 队列 set set set 执行 -------------------
事务中每条命令都会被序列化，执行过程中按顺序执行，不允许其他命令进行干扰。
一次性
顺序性
排他性
Redis事务没有隔离级别的概念
Redis单条命令是保证原子性的，但是事务不保证原子性！
```



Redis事务操作过程

- 开启事务（multi）
- 命令入队
- 执行事务（exec）

所以事务中的命令在加入时都没有被执行，直到提交时才会开始执行(Exec)一次性完成。

## 正常执行



```
127.0.0.1:6379> multi # 开启事务
OK
127.0.0.1:6379> set k1 v1 # 命令入队
QUEUED
127.0.0.1:6379> set k2 v2 # ..
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> keys *
QUEUED
127.0.0.1:6379> exec # 事务执行
1) OK
2) OK
3) "v1"
4) OK
5) 1) "k3"
   2) "k2"
   3) "k1"
```

## 取消事务(discurd)



```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> DISCARD # 放弃事务
OK
127.0.0.1:6379> EXEC 
(error) ERR EXEC without MULTI # 当前未开启事务
127.0.0.1:6379> get k1 # 被放弃事务中命令并未执行
(nil)
```

## 事务错误

> 代码语法错误（编译时异常）所有的命令都不执行



```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> error k1 # 这是一条语法错误命令
(error) ERR unknown command `error`, with args beginning with: `k1`, # 会报错但是不影响后续命令入队 
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> EXEC
(error) EXECABORT Transaction discarded because of previous errors. # 执行报错
127.0.0.1:6379> get k1 
(nil) # 其他命令并没有被执行
```

> 代码逻辑错误 (运行时异常) **其他命令可以正常执行 ** >>> 所以不保证事务原子性



```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> INCR k1 # 这条命令逻辑错误（对字符串进行增量）
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) (error) ERR value is not an integer or out of range # 运行时报错
4) "v2" # 其他命令正常执行
虽然中间有一条命令报错了，但是后面的指令依旧正常执行成功了。
所以说Redis单条指令保证原子性，但是Redis事务不能保证原子性。
```



**监控**
悲观锁：

- 很悲观，认为什么时候都会出现问题，无论做什么都会加锁

乐观锁：

- 很乐观，认为什么时候都不会出现问题，所以不会上锁！更新数据的时候去判断一下，在此期间是否有人修改过这个数据
- 获取version
- 更新的时候比较version

使用watch key监控指定数据，相当于乐观锁加锁。

> 正常执行



```
127.0.0.1:6379> set money 100 # 设置余额:100
OK
127.0.0.1:6379> set use 0 # 支出使用:0
OK
127.0.0.1:6379> watch money # 监视money (上锁)
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY use 20
QUEUED
127.0.0.1:6379> exec # 监视值没有被中途修改，事务正常执行
1) (integer) 80
2) (integer) 20
```

测试多线程修改值，使用watch可以当做redis的乐观锁操作（相当于getversion）

我们启动另外一个客户端模拟插队线程。

线程1：



```
127.0.0.1:6379> watch money # money上锁
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY use 20
QUEUED
127.0.0.1:6379>     # 此时事务并没有执行
```

模拟线程插队，线程2：



```
127.0.0.1:6379> INCRBY money 500 # 修改了线程一中监视的money
(integer) 600
```

回到线程1，执行事务：



```
127.0.0.1:6379> EXEC # 执行之前，另一个线程修改了我们的值，这个时候就会导致事务执行失败
(nil) # 没有结果，说明事务执行失败
127.0.0.1:6379> get money # 线程2 修改生效
"600"
127.0.0.1:6379> get use # 线程1事务执行失败，数值没有被修改
"0"
```



解锁获取最新值，然后再加锁进行事务。

unwatch进行解锁。

注意：每次提交执行exec后都会自动释放锁，不管是否成功

# 六、Jedis

使用Java来操作Redis，Jedis是Redis官方推荐使用的Java连接redis的客户端。

1.导入依赖



```
<!--导入jredis的包-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
<!--fastjson-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.70</version>
</dependency>
```

2.编码测试

> 连接数据库
> 操作命令
> 断开连接

代码示例



```
public class TestPing {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("192.168.xx.xxx", 6379);
        String response = jedis.ping();
        System.out.println(response); // PONG
    }
}
```

输出PONG

## 常用的API

string、list、set、hash、zset

所有的api命令，就是我们对应的上面学习的指令，一个都没有变化！



```
public class TestTX {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello","world");
        jsonObject.put("name","kuangshen");
        // 开启事务
        Transaction multi = jedis.multi();
        String result = jsonObject.toJSONString();
        // jedis.watch(result)
        try {
            multi.set("user1",result);
            multi.set("user2",result);
            int i = 1/0 ; // 代码抛出异常事务，执行失败！
            multi.exec(); // 执行事务！
        } catch (Exception e) {
            multi.discard(); // 放弃事务
            e.printStackTrace();
        } finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close(); // 关闭连接
        }
    }
}
```

# 七、SpringBoot整合

SpringBoot 操作数据：spring-data jpa jdbc mongodb redis！

SpringData 也是和 SpringBoot 齐名的项目！

说明： 在 SpringBoot2.x 之后，原来使用的jedis 被替换为了 lettuce?

jedis : 采用的直连，多个线程操作的话，是不安全的，如果想要避免不安全的，使用 jedis pool 连接池！ 更像 BIO 模式

lettuce : 采用netty，实例可以再多个线程中进行共享，不存在线程不安全的情况！可以减少线程数据了，更像 NIO 模式

源码分析：



```
@Bean
@ConditionalOnMissingBean(name = "redisTemplate") 
// 我们可以自己定义一个redisTemplate来替换这个默认的！
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
// 默认的 RedisTemplate 没有过多的设置，redis 对象都是需要序列化！
// 两个泛型都是 Object, Object 的类型，我们后使用需要强制转换 <String, Object>
    RedisTemplate<Object, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
@Bean
@ConditionalOnMissingBean // 由于 String 是redis中最常使用的类型，所以说单独提出来了一个bean！
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```

## 整合测试

1.导入依赖



```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

springboot 2.x后 ，原来使用的 Jedis 被 lettuce 替换。

jedis：采用的直连，多个线程操作的话，是不安全的。如果要避免不安全，使用jedis pool连接池！更像BIO模式

lettuce：采用netty，实例可以在多个线程中共享，不存在线程不安全的情况！可以减少线程数据了，更像NIO模式

我们在学习SpringBoot自动配置的原理时，整合一个组件并进行配置一定会有一个自动配置类xxxAutoConfiguration,并且在spring.factories中也一定能找到这个类的完全限定名。Redis也不例外。

[![img](https://img-blog.csdnimg.cn/20200513214531573.png)](https://img-blog.csdnimg.cn/20200513214531573.png)

那么就一定还存在一个RedisProperties类

[![img](https://img-blog.csdnimg.cn/20200513214607475.png)](https://img-blog.csdnimg.cn/20200513214607475.png)

@ConditionalOnClass注解中有两个类是默认不存在的，所以Jedis是无法生效的

然后再看Lettuce：

[![img](https://img-blog.csdnimg.cn/20200513214618179.png)](https://img-blog.csdnimg.cn/20200513214618179.png)

完美生效。

现在我们回到RedisAutoConfiguratio

[![img](https://img-blog.csdnimg.cn/2020051321462777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/2020051321462777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

只有两个简单的Bean

- RedisTemplate
- StringRedisTemplate

当看到xxTemplate时可以对比RestTemplat、SqlSessionTemplate,通过使用这些Template来间接操作组件。那么这俩也不会例外。分别用于操作Redis和Redis中的String数据类型。

在RedisTemplate上也有一个条件注解，说明我们是可以对其进行定制化的

说完这些，我们需要知道如何编写配置文件然后连接Redis，就需要阅读RedisProperties

[![img](https://img-blog.csdnimg.cn/20200513214638238.png)](https://img-blog.csdnimg.cn/20200513214638238.png)

这是一些基本的配置属性。

[![img](https://img-blog.csdnimg.cn/20200513214649380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513214649380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

还有一些连接池相关的配置。注意使用时一定使用**Lettuce**的连接池。

[![img](https://img-blog.csdnimg.cn/20200513214700372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513214700372.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

2.编写配置文件



```
# 配置redis
spring.redis.host=39.99.xxx.xx
spring.redis.port=6379
```

3.使用RedisTemplate





@SpringBootTest



class Redis02SpringbootApplicationTests {



```
@Autowired
private RedisTemplate redisTemplate;

@Test
void contextLoads() {

    // redisTemplate 操作不同的数据类型，api和我们的指令是一样的
    // opsForValue 操作字符串 类似String
    // opsForList 操作List 类似List
    // opsForSet
    // opsForHash
    // opsForZSet
    // opsForGeo
    // opsForHyperLog

    // 除了基本的操作，我们常用的方法都可以直接通过redisTemplate操作，比如事务和基本的CRUD

    // 获取连接对象
    //RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
    //connection.flushDb();
    //connection.flushAll();

    redisTemplate.opsForValue().set(&quot;mykey&quot;,&quot;kuangshen&quot;);
    System.out.println(redisTemplate.opsForValue().get(&quot;mykey&quot;));
}
```



}



4.测试结果

此时我们回到Redis查看数据时候，惊奇发现全是乱码，可是程序中可以正常输出。这时候就关系到存储对象的序列化问题，在网络中传输的对象也是一样需要序列化，否者就全是乱码。

RedisTemplate内部的序列化配置是这样的

[![img](https://img-blog.csdnimg.cn/20200513214746506.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513214746506.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

默认的序列化器是采用JDK序列化器

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B0.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记.jpg?raw=true)

后续我们定制RedisTemplate就可以对其进行修改。

RedisSerializer提供了多种序列化方案：

[![img](https://img-blog.csdnimg.cn/20200513214818682.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513214818682.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

我们来编写一个自己的 RedisTemplete

``` java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
@Configuration
public class RedisConfig {
    // 这是我给大家写好的一个固定模板，大家在企业中，拿去就可以直接使用！
    // 自己定义了一个 RedisTemplate
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 我们为了自己开发方便，一般直接使用 <String, Object>
        RedisTemplate<String, Object> template = new RedisTemplate<String,
        Object>();
        template.setConnectionFactory(factory);
    // Json序列化配置
    Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new
    Jackson2JsonRedisSerializer(Object.class);
    ObjectMapper om = new ObjectMapper();
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    jackson2JsonRedisSerializer.setObjectMapper(om);
    // String 的序列化
    StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

    // key采用String的序列化方式
    template.setKeySerializer(stringRedisSerializer);
    // hash的key也采用String的序列化方式
    template.setHashKeySerializer(stringRedisSerializer);
    // value序列化方式采用jackson
    template.setValueSerializer(jackson2JsonRedisSerializer);
    // hash的value序列化方式采用jackson
    template.setHashValueSerializer(jackson2JsonRedisSerializer);
    template.afterPropertiesSet();

    return template;
	}

}



```

所有的redis操作，其实对于java开发人员来说，十分的简单，更重要是要去理解redis的思想和每一种数据结构的用处和作用场景！

# 八、Redis.conf

## 容量单位不区分大小写，G和GB有区别

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B002.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记02.jpg?raw=true)

## 可以使用 include 组合多个配置问题

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B003.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记03.jpg?raw=true)

## 网络配置

[![img](https://img-blog.csdnimg.cn/20200513214912813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513214912813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

## 日志



```
# 日志
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 生产环境
# warning (only very important / critical messages are logged)
loglevel notice
logfile "" # 日志的文件位置名
databases 16 # 数据库的数量，默认是 16 个数据库
always-show-logo yes # 是否总是显示LOGO
```

> 日志输出级别
>
> > debug
> > verbose
> > notice
> > waring

## 持久化规则

持久化， 在规定的时间内，执行了多少次操作，则会持久化到文件 .rdb. aof redis 是内存数据库，如果没有持久化，那么数据断电及失！

由于Redis是基于内存的数据库，需要将数据由内存持久化到文件中

持久化方式：

RDB
AOF



```
# 如果900s内，如果至少有一个1 key进行了修改，我们及进行持久化操作
save 900 1
# 如果300s内，如果至少10 key进行了修改，我们及进行持久化操作
save 300 10
# 如果60s内，如果至少10000 key进行了修改，我们及进行持久化操作
save 60 10000
# 我们之后学习持久化，会自己定义这个测试！
stop-writes-on-bgsave-error yes # 持久化如果出错，是否还需要继续工作！
rdbcompression yes # 是否压缩 rdb 文件，需要消耗一些cpu资源！
rdbchecksum yes # 保存rdb文件的时候，进行错误的检查校验！
dir ./ # rdb 文件保存的目录！
```

## SECURITY 安全

可以在这里设置redis的密码，默认是没有密码！



```
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> config get requirepass # 获取redis的密码
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass "123456" # 设置redis的密码
OK
127.0.0.1:6379> config get requirepass # 发现所有的命令都没有权限了
(error) NOAUTH Authentication required.
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456 # 使用密码进行登录！
OK
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
```

## 限制 CLIENTS（客户端连接相关）



```
maxclients 10000 # 设置能连接上redis的最大客户端的数量
maxmemory <bytes> # redis 配置最大的内存容量
maxmemory-policy noeviction # 内存到达上限之后的处理策略
1、volatile-lru：只对设置了过期时间的key进行LRU（默认值）
2、allkeys-lru ： 删除lru算法的key
3、volatile-random：随机删除即将过期key
4、allkeys-random：随机删除
5、volatile-ttl ： 删除即将过期的
6、noeviction ： 永不过期，返回错误
```

## APPEND ONLY 模式 aof配置



```
appendonly no # 默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分所有的情况下，
rdb完全够用！
appendfilename "appendonly.aof" # 持久化的文件的名字
# appendfsync always # 每次修改都会 sync。消耗性能
appendfsync everysec # 每秒执行一次 sync，可能会丢失这1s的数据！
# appendfsync no # 不执行 sync，这个时候操作系统自己同步数据，速度最快！
```

# 九、Redis持久化——RDB

面试和工作，持久化都是重点！
Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了持久化功能！

## 什么是RDB（Redis DataBase）

> 在指定时间间隔后，将内存中的数据集快照写入数据库 ；在恢复时候，直接读取快照文件，进行数据的恢复 ；
> 默认情况下， Redis 将数据库快照保存在名字为 dump.rdb的二进制文件中。文件名可以在配置文件中进行自定义。

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。
Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。
这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。我们默认的就是RDB，一般情况下不需要修改这个配置！ 有时候在生产环境我们会将这个文件进行备份！
rdb保存的文件是dump.rdb 都是在我们的配置文件中快照中进行配置的！

## 工作原理

在进行 RDB 的时候，redis 的主线程是不会做 io 操作的，主线程会 fork 一个子线程来完成该操作；

1. Redis 调用forks。同时拥有父进程和子进程。
2. 子进程将数据集写入到一个临时 RDB 文件中。
3. 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

这种工作方式使得 Redis 可以从写时复制（copy-on-write）机制中获益(因为是使用子进程进行写操作，而父进程依然可以接收来自客户端的请求。)

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B004.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记04.jpg?raw=true)

## 触发机制

1. save的规则满足的情况下，会自动触发rdb原则
2. 执行flushall命令，也会触发我们的rdb原则
3. 退出redis，也会自动产生rdb文件

### save

使用 save 命令，会立刻对当前内存中的数据进行持久化 ,但是会阻塞，也就是不接受其他操作了；

> 由于 save 命令是同步命令，会占用Redis的主进程。若Redis数据非常多时，save命令执行速度会非常慢，阻塞所有客户端的请求。

示意图

[![img](https://img-blog.csdnimg.cn/20200513215150892.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215150892.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

### flushall命令

flushall 命令也会触发持久化 ；

触发持久化规则
满足配置条件中的触发条件 ；

> 可以通过配置文件对 Redis 进行设置， 让它在“ N 秒内数据集至少有 M 个改动”这一条件被满足时， 自动进行数据集保存操作。

[![img](https://img-blog.csdnimg.cn/20200513215205970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215205970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

[![img](https://img-blog.csdnimg.cn/20200513215220858.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215220858.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

### bgsave

bgsave 是异步进行，进行持久化的时候，redis 还可以将继续响应客户端请求 ；

[![img](https://img-blog.csdnimg.cn/2020051321523151.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/2020051321523151.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

bgsave和save对比

| 命令   | save               | bgsave                             |
| :----- | :----------------- | :--------------------------------- |
| IO类型 | 同步               | 异步                               |
| 阻塞？ | 是                 | 是（阻塞发生在fock()，通常非常快） |
| 复杂度 | O(n)               | O(n)                               |
| 优点   | 不会消耗额外的内存 | 不阻塞客户端命令                   |
| 缺点   | 阻塞客户端命令     | 需要fock子进程，消耗内存           |

## 如果恢复rdb文件！

1、只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检查dump.rdb 恢复其中的数据！
2、查看需要存在的位置



```
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/bin" # 如果在这个目录下存在 dump.rdb 文件，启动就会自动恢复其中的数据
```

## 优缺点

> 优点：

1. 适合大规模的数据恢复
2. 对数据的完整性要求不高

> 缺点：

1. 需要一定的时间间隔进行操作，如果redis意外宕机了，这个最后一次修改的数据就没有了。
2. fork进程的时候，会占用一定的内容空间。

# 十、Redis持久化——AOF

Append Only File

将我们所有的命令都记录下来，history，恢复的时候就把这个文件全部再执行一遍

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B005.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记05.jpg?raw=true)

以日志的形式来记录每个写的操作，将Redis执行过的所有指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

## 什么是AOF

 快照功能（RDB）并不是非常耐久（durable）： 如果 Redis 因为某些原因而造成故障停机， 那么服务器将丢失最近写入、以及未保存到快照中的那些数据。 从 1.1 版本开始， Redis 增加了一种完全耐久的持久化方式： AOF 持久化。

appendonly no yes则表示启用AOF

默认是不开启的，我们需要手动配置，然后重启redis，就可以生效了！

如果这个aof文件有错位，这时候redis是启动不起来的，我需要修改这个aof文件

redis给我们提供了一个工具redis-check-aof --fix



```
appendonly yes  # 默认是不开启aof模式的，默认是使用rdb方式持久化的，在大部分的情况下，rdb完全够用
appendfilename "appendonly.aof"
appendfsync always # 每次修改都会sync 消耗性能
appendfsync everysec # 每秒执行一次 sync 可能会丢失这一秒的数据
appendfsync no # 不执行 sync ,这时候操作系统自己同步数据，速度最快
```



## 优缺点

优点

1. 每一次修改都会同步，文件的完整性会更加好
2. 没秒同步一次，可能会丢失一秒的数据
3. 从不同步，效率最高

缺点

1. 相对于数据文件来说，aof远远大于rdb，修复速度比rdb慢！
2. Aof运行效率也要比rdb慢，所以我们redis默认的配置就是rdb持久化

## 扩展：

1、RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2、AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化
4、同时开启两种持久化方式

- 在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
- RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？作者建议不要，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。

5、性能建议

- 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
- 如果Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
- 如果不Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构。

## 如何选择使用哪种持久化方式？

一般来说， 如果想达到足以媲美 PostgreSQL 的数据安全性， 你应该同时使用两种持久化功能。

如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失， 那么你可以只使用 RDB 持久化。

有很多用户都只使用 AOF 持久化， 但并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快。

# 十一、Redis发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。微信、 微博、关注系统！ Redis 客户端可以订阅任意数量的频道。 订阅/发布消息图： 第一个：消息发送者， 第二个：频道 第三个：消息订阅者！

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B006.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记06.jpg?raw=true)

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

[![img](https://img-blog.csdnimg.cn/20200513215523258.png)](https://img-blog.csdnimg.cn/20200513215523258.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

[![img](https://img-blog.csdnimg.cn/2020051321553483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/2020051321553483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

## 命令

- `PSUBSCRIBE pattern [pattern..]` 订阅一个或多个符合给定模式的频道。
- `PUNSUBSCRIBE pattern [pattern..]` 退订一个或多个符合给定模式的频道。
- `PUBSUB subcommand [argument[argument]]` 查看订阅与发布系统状态。
- `PUBLISH channel message` 向指定频道发布消息
- `SUBSCRIBE channel [channel..]` 订阅给定的一个或多个频道。
- `UNSUBSCRIBE channel [channel..]` 退订一个或多个频道

代码示例



```
------------订阅端----------------------
127.0.0.1:6379> SUBSCRIBE sakura # 订阅sakura频道
Reading messages... (press Ctrl-C to quit) # 等待接收消息
1) "subscribe" # 订阅成功的消息
2) "sakura"
3) (integer) 1
1) "message" # 接收到来自sakura频道的消息 "hello world"
2) "sakura"
3) "hello world"
1) "message" # 接收到来自sakura频道的消息 "hello i am sakura"
2) "sakura"
3) "hello i am sakura"
--------------消息发布端-------------------
127.0.0.1:6379> PUBLISH sakura "hello world" # 发布消息到sakura频道
(integer) 1
127.0.0.1:6379> PUBLISH sakura "hello i am sakura" # 发布消息
(integer) 1
-----------------查看活跃的频道------------
127.0.0.1:6379> PUBSUB channels

"sakura"
```

## 原理

Redis是使用C实现的，通过分析 Redis 源码里的 pubsub.c 文件，了解发布和订阅机制的底层实现，籍此加深对 Redis 的理解。

Redis 通过 PUBLISH 、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。

每个 Redis 服务器进程都维持着一个表示服务器状态的 redis.h/redisServer 结构， 结构的 pubsub_channels 属性是一个字典， 这个字典就用于保存订阅频道的信息，其中，字典的键为正在被订阅的频道， 而字典的值则是一个链表， 链表中保存了所有订阅这个频道的客户端。

[![img](https://img-blog.csdnimg.cn/2020051321554964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/2020051321554964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

客户端订阅，就被链接到对应频道的链表的尾部，退订则就是将客户端节点从链表中移除。

## 缺点

1. 如果一个客户端订阅了频道，但自己读取消息的速度却不够快的话，那么不断积压的消息会使redis输出缓冲区的体积变得越来越大，这可能使得redis本身的速度变慢，甚至直接崩溃。
2. 这和数据传输可靠性有关，如果在订阅方断线，那么他将会丢失所有在短线期间发布者发布的消息。

## 应用

1. 消息订阅：公众号订阅，微博关注等等（起始更多是使用消息队列来进行实现）
2. 多人在线聊天室。

稍微复杂的场景，我们就会使用消息中间件MQ处理。

# 十二、Redis主从复制

## 概念

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点（Master/Leader）,后者称为从节点（Slave/Follower）， 数据的复制是单向的！只能由主节点复制到从节点（主节点以写为主、从节点以读为主）。

默认情况下，每台Redis服务器都是主节点，一个主节点可以有0个或者多个从节点，但每个从节点只能由一个主节点。

## 作用

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余的方式。
2. 故障恢复：当主节点故障时，从节点可以暂时替代主节点提供服务，是一种服务冗余的方式
3. 负载均衡：在主从复制的基础上，配合读写分离，由主节点进行写操作，从节点进行读操作，分担服务器的负载；尤其是在多读少写的场景下，通过多个从节点分担负载，提高并发量。
4. 高可用基石：主从复制还是哨兵和集群能够实施的基础。

## 为什么使用集群

一般来说，要将Redis运用于工程项目中，只使用一台Redis是万万不能的（宕机），原因如下：

1、从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大；

2、从容量上，单个Redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有内存用作Redis存储内存，一般来说，单台Redis最大使用内存不应该超过20G。

电商网站上的商品，一般都是一次上传，无数次浏览的，说专业点也就是"多读少写"。

对于这种场景，我们可以使如下这种架构：

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B007.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记07.jpg?raw=true)

主从复制，读写分离！ 80% 的情况下都是在进行读操作！减缓服务器的压力！架构中经常使用！ 一主二从！

只要在公司中，主从复制就是必须要使用的，因为在真实的项目中不可能单机使用Redis！

**总结**

1. 单台服务器难以负载大量的请求
2. 单台服务器故障率高，系统崩坏概率大
3. 单台服务器内存容量有限。

## 环境配置

只配置从库，不用配置主库！



```
127.0.0.1:6379> info replication
# Replication
role:master # 角色
connected_slaves:0 # 从机数量
master_replid:3b54deef5b7b7b7f7dd8acefa23be48879b4fcff
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

复制3个配置文件，然后修改对应的信息

1. 端口
2. pid名字
3. log文件名
4. dump.rdb名字

启动单机多服务集群：

[![img](https://img-blog.csdnimg.cn/20200513215610163.png)](https://img-blog.csdnimg.cn/20200513215610163.png)

## 一主二从配置

默认情况下，每台Redis服务器都是主节点；我们一般情况下只用配置从机就好了！

认老大！一主（79）二从（80，81）

使用SLAVEOF host port就可以为从机配置主机了。

[![img](https://img-blog.csdnimg.cn/20200513215637483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215637483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

**说明**

- `SLAVEOF host 6379` 找谁当自己的老大！
- `role:slave` # 当前角色是从机
- `master_host:127.0.0.1` # 可以的看到主机的信息

然后主机上也能看到从机的状态：

[![img](https://img-blog.csdnimg.cn/20200513215645778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215645778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

**说明**

- `connected_slaves:1` # 多了从机的配置
- `slave0:ip=127.0.0.1,port=6380,state=online,offset=42,lag=1` # 多了从机的配置

真实的从主配置应该在配置文件中配置，这样的话是永久的，我们这里使用的是命令，暂时的！

## 使用规则

1.从机只能读，不能写，主机可读可写但是多用于写。



```
127.0.0.1:6381> set name sakura # 从机6381写入失败
(error) READONLY You can't write against a read only replica.
127.0.0.1:6380> set name sakura # 从机6380写入失败
(error) READONLY You can't write against a read only replica.
127.0.0.1:6379> set name sakura
OK
127.0.0.1:6379> get name
"sakura"
```



2.当主机断电宕机后，默认情况下从机的角色不会发生变化 ，集群中只是失去了写操作，当主机恢复以后，又会连接上从机恢复原状。

3.当从机断电宕机后，若不是使用配置文件配置的从机，再次启动后作为主机是无法获取之前主机的数据的，若此时重新配置称为从机，又可以获取到主机的所有数据。这里就要提到一个同步原理。

4.第二条中提到，默认情况下，主机故障后，不会出现新的主机，有两种方式可以产生新的主机：

- 从机手动执行命令slaveof no one,这样执行以后从机会独立出来成为一个主机
- 使用哨兵模式（自动选举）

> 如果没有老大了，这个时候能不能选择出来一个老大呢？手动！

如果主机断开了连接，我们可以使用`SLAVEOF no one`让自己变成主机！其他的节点就可以手动连接到最新的主节点（手动）！如果这个时候老大修复了，那么就重新连接！

## 复制原理

Slave 启动成功连接到 master 后会发送一个sync同步命令

Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。

**全量复制**：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

**增量复制**：Master 继续将新的所有收集到的修改命令依次传给slave，完成同步

但是只要是重新连接master，一次完全同步（全量复制）将被自动执行！ 我们的数据一定可以在从机中看到！

# 十三、哨兵模式

（自动选举老大的模式）

## 概述

主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。Redis从2.8开始正式提供了Sentinel（哨兵） 架构来解决这个问题。**能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。**

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B008.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记08.jpg?raw=true)

哨兵的作用：

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了**多哨兵模式**。

[![img](https://github.com/Atomy5011/blog-picture/blob/master/%E7%8B%82%E7%A5%9E%E8%AF%B4Redis%E7%AC%94%E8%AE%B009.jpg?raw=true)](https://github.com/Atomy5011/blog-picture/blob/master/狂神说Redis笔记09.jpg?raw=true)

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为主观下线。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover[故障转移]操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。

## 测试

1、配置哨兵配置文件 sentinel.conf



```
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
```

后面的这个数字1，代表主机挂了，slave投票看让谁接替成为主机，票数最多的，就会成为主机！

2、启动哨兵！

[![img](https://img-blog.csdnimg.cn/20200513215752444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215752444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

3、此时哨兵监视着我们的主机6379，当我们断开主机后：

[![img](https://img-blog.csdnimg.cn/20200513215806972.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215806972.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

## 哨兵模式优缺点

优点：

1. 哨兵集群，基于主从复制模式，所有主从复制的优点，它都有
2. 主从可以切换，故障可以转移，系统的可用性更好
3. 哨兵模式是主从模式的升级，手动到自动，更加健壮 缺点：
4. Redis不好在线扩容，集群容量一旦达到上限，在线扩容就十分麻烦
5. 实现哨兵模式的配置其实是很麻烦的，里面有很多配置项

## 哨兵模式的全部配置

完整的哨兵模式配置文件 sentinel.conf



```
# Example sentinel.conf
哨兵sentinel实例运行的端口 默认26379
port 26379
哨兵sentinel的工作目录
dir /tmp
哨兵sentinel监控的redis主节点的 ip port
master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了
sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 1
当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
这个数字越小，完成failover所需的时间就越长，
但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
故障转移的超时时间 failover-timeout 可以用在以下这些方面：
1. 同一个sentinel对同一个master两次failover之间的间隔时间。
2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
3.当想要取消一个正在进行的failover所需要的时间。
4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
默认三分钟
sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
SCRIPTS EXECUTION
配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
对于脚本的运行结果有以下规则：
若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，
一个是事件的类型，
一个是事件的描述。
如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
通知脚本
sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh
客户端重新配置主节点参数脚本
当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
以下参数将会在调用脚本时传给脚本:
<master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
目前<state>总是“failover”,
<role>是“leader”或者“observer”中的一个。
参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
这个脚本应该是通用的，能被多次调用，不是针对性的。
sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```



# 十四、缓存穿透与雪崩

## 缓存穿透（即查询不到）

### 概念

在默认情况下，用户请求数据时，会先在缓存(Redis)中查找，若没找到即缓存未命中，再在数据库中进行查找，数量少可能问题不大，可是一旦大量的请求数据（例如秒杀场景）缓存都没有命中的话，就会全部转移到数据库上，造成数据库极大的压力，就有可能导致数据库崩溃。网络安全中也有人恶意使用这种手段进行攻击被称为洪水攻击。

### 解决方案

**布隆过滤器**

对所有可能查询的参数以Hash的形式存储，以便快速确定是否存在这个值，在控制层先进行拦截校验，校验不通过直接打回，减轻了存储系统的压力。

[![img](https://img-blog.csdnimg.cn/20200513215824722.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215824722.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

**缓存空对象**

一次请求若在缓存和数据库中都没找到，就在缓存中方一个空对象用于处理后续这个请求。

[![img](https://img-blog.csdnimg.cn/20200513215836317.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215836317.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

这样做有一个缺陷：存储空对象也需要空间，大量的空对象会耗费一定的空间，存储效率并不高。解决这个缺陷的方式就是设置较短过期时间

即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。

## 缓存击穿（即量太大，缓存过期）

### 概念

相较于缓存穿透，缓存击穿的目的性更强，一个存在的key，在缓存过期的一刻，同时有大量的请求，这些请求都会击穿到DB，造成瞬时DB请求量大、压力骤增。这就是缓存被击穿，只是针对其中某个key的缓存不可用而导致击穿，但是其他的key依然可以使用缓存响应。

 比如热搜排行上，一个热点新闻被同时大量访问就可能导致缓存击穿。

### 解决方案

1.设置热点数据永不过期

这样就不会出现热点数据过期的情况，但是当Redis内存空间满的时候也会清理部分数据，而且此种方案会占用空间，一旦热点数据多了起来，就会占用部分空间。

2.加互斥锁(分布式锁)

在访问key之前，采用SETNX（set if not exists）来设置另一个短期key来锁住当前key的访问，访问结束再删除该短期key。保证同时刻只有一个线程访问。这样对锁的要求就十分高。

## 缓存雪崩

### 概念

大量的key设置了相同的过期时间，导致在缓存在同一时刻全部失效，造成瞬时DB请求量大、压力骤增，引起雪崩。

缓存雪崩，是指在某一个时间段，缓存集中过期失效。Redis 宕机！

产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。

[![img](https://img-blog.csdnimg.cn/20200513215850428.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20200513215850428.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg3MzIyNw==,size_16,color_FFFFFF,t_70)

其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。

### 解决方案

> redis高可用

这个思想的含义是，既然redis有可能挂掉，那我多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群

> 限流降级

这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

> 数据预热

数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。

# 十五、淘汰策略

Redis的数据已经设置了TTL，不是过期就已经删除了吗？为什么还存在所谓的淘汰策略呢？这个原因我们需要从redis的过期策略聊起。

## 1.两种淘汰策略

### 	1.1、定期删除

redis 会将每个设置了过期时间的 key 放入到一个独立的字典中，以后会定期遍历这个字典来删除到期的 key。

Redis 默认会每秒进行十次过期扫描（100ms一次），过期扫描不会遍历过期字典中所有的 key，而是采用了一种简单的贪心策略。

1.从过期字典中随机 20 个 key；

2.删除这 20 个 key 中已经过期的 key；

3.如果过期的 key 比率超过 1/4，那就重复步骤 1；

redis默认是每隔 100ms就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除。注意这里是随机抽取的。为什么要随机呢？你想一想假如 redis 存了几十万个 key ，每隔100ms就遍历所有的设置过期时间的 key 的话，就会给 CPU 带来很大的负载。

### 1.2、惰性删除

所谓惰性策略就是在客户端访问这个key的时候，redis对key的过期时间进行检查，如果过期了就立即删除，不会给你返回任何东西。

定期删除可能会导致很多过期key到了时间并没有被删除掉。所以就有了惰性删除。假如你的过期 key，靠定期删除没有被删除掉，还停留在内存里，除非你的系统去查一下那个 key，才会被redis给删除掉。这就是所谓的惰性删除，即当你主动去查过期的key时,如果发现key过期了,就立即进行删除,不返回任何东西.

总结：定期删除是集中处理，惰性删除是零散处理。

## 2.为什么需要淘汰策略

有了以上过期策略的说明后，就很容易理解为什么需要淘汰策略了，因为不管是定期采样删除还是惰性删除都不是一种完全精准的删除，就还是会存在key没有被删除掉的场景，所以就需要内存淘汰策略进行补充。

### 2.1、内存淘汰策略

1. noeviction：当内存使用超过配置的时候会返回错误，不会驱逐任何键

2. allkeys-lru：加入键的时候，如果过限，首先通过LRU算法驱逐最久没有使用的键

3. volatile-lru：加入键的时候如果过限，首先从设置了过期时间的键集合中驱逐最久没有使用的键

4. allkeys-random：加入键的时候如果过限，从所有key随机删除

5. volatile-random：加入键的时候如果过限，从过期键的集合中随机驱逐

6. volatile-ttl：从配置了过期时间的键中驱逐马上就要过期的键

7. volatile-lfu：从所有配置了过期时间的键中驱逐使用频率最少的键

8. allkeys-lfu：从所有键中驱逐使用频率最少的键

## 3、LRU

### 3.1、标准LRU实现方式

![img](https://pic1.zhimg.com/80/v2-9374e1c1ef6c7c8d14491b4092d79f80_720w.jpg)

1. 新增key value的时候首先在链表结尾添加Node节点，如果超过LRU设置的阈值就淘汰队头的节点并删除掉HashMap中对应的节点。

2. 修改key对应的值的时候先修改对应的Node中的值，然后把Node节点移动队尾。

3. 访问key对应的值的时候把访问的Node节点移动到队尾即可。

### 3.2、Redis的LRU实现

Redis维护了一个24位时钟，可以简单理解为当前系统的时间戳，每隔一定时间会更新这个时钟。每个key对象内部同样维护了一个24位的时钟，当新增key对象的时候会把系统的时钟赋值到这个内部对象时钟。比如我现在要进行LRU，那么首先拿到当前的全局时钟，然后再找到内部时钟与全局时钟距离时间最久的（差最大）进行淘汰，这里值得注意的是全局时钟只有24位，按秒为单位来表示才能存储194天，所以可能会出现key的时钟大于全局时钟的情况，如果这种情况出现那么就两个相加而不是相减来求最久的key。

```cpp
struct redisServer {
       pid_t pid; 
       char *configfile; 
       //全局时钟
       unsigned lruclock:LRU_BITS; 
       ...
};
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    /* key对象内部时钟 */
    unsigned lru:LRU_BITS;
    int refcount;
    void *ptr;
} robj;
```

Redis中的LRU与常规的LRU实现并不相同，常规LRU会准确的淘汰掉队头的元素，但是Redis的LRU并不维护队列，只是根据配置的策略要么从所有的key中随机选择N个（N可以配置）要么从所有的设置了过期时间的key中选出N个键，然后再从这N个键中选出最久没有使用的一个key进行淘汰。

下图是常规LRU淘汰策略与Redis随机样本取一键淘汰策略的对比，浅灰色表示已经删除的键，深灰色表示没有被删除的键，绿色表示新加入的键，越往上表示键加入的时间越久。从图中可以看出，在redis 3中，设置样本数为10的时候能够很准确的淘汰掉最久没有使用的键，与常规LRU基本持平。

![img](https://pic4.zhimg.com/80/v2-e11b852bc091817560ec62b7082995db_720w.jpg)

### 3.3、为什么要使用近似LRU？

1、性能问题，由于近似LRU算法只是最多随机采样N个key并对其进行排序，如果精准需要对所有key进行排序，这样近似LRU性能更高

2、内存占用问题，redis对内存要求很高，会尽量降低内存使用率，如果是抽样排序可以有效降低内存的占用

3、实际效果基本相等，如果请求符合长尾法则，那么真实LRU与Redis LRU之间表现基本无差异

4、在近似情况下提供可自配置的取样率来提升精准度，例如通过 CONFIG SET maxmemory-samples <count> 指令可以设置取样数，取样数越高越精准，如果你的CPU和内存有足够，可以提高取样数看命中率来探测最佳的采样比例。

## 4、LFU

LFU是在Redis4.0后出现的，LRU的最近最少使用实际上并不精确，考虑下面的情况，如果在|处删除，那么A距离的时间最久，但实际上A的使用频率要比B频繁，所以合理的淘汰策略应该是淘汰B。LFU就是为应对这种情况而生的。

A~~A~~A~~A~~A~~A~~A~~A~~A~~A~~~|

B~~~~~B~~~~~B~~~~~B~~~~~~~~~~~~B|

LFU把原来的key对象的内部时钟的24位分成两部分，前16位还代表时钟，后8位代表一个计数器。16位的情况下如果还按照秒为单位就会导致不够用，所以一般这里以时钟为单位。而后8位表示当前key对象的访问频率，8位只能代表255，但是redis并没有采用线性上升的方式，而是通过一个复杂的公式，通过配置如下两个参数来调整数据的递增速度。

lfu-log-factor 可以调整计数器counter的增长速度，lfu-log-factor越大，counter增长的越慢。

lfu-decay-time 是一个以分钟为单位的数值，可以调整counter的减少速度。

所以这两个因素就对应到了LFU的Counter减少策略和增长策略，它们实现逻辑分别如下。

### 4.1、降低LFUDecrAndReturn

1. 先从高16位获取最近的降低时间ldt以及低8位的计数器counter值
2. 计算当前时间now与ldt的差值（now-ldt），当ldt大于now时，那说明是过了一个周期，按照65535-ldt+now计算（16位一个周期最大65535）
3. 使用第2步计算的差值除以lfu_decay_time，即LFUTimeElapsed(ldt) / server.lfu_decay_time，已过去n个lfu_decay_time，则将counter减少n。

### 4.2、增长LFULogIncr

1、获取0-1的随机数r

2、计算0-1之间的控制因子p，它的计算逻辑如下

```cpp
//LFU_INIT_VAL默认为5
baseval = counter - LFU_INIT_VAL;
//计算控制因子
p = 1.0/(baseval*lfu_log_factor+1);
```

3、如果r小于p，counter增长1

p取决于当前counter值与lfu_log_factor因子，counter值与lfu_log_factor因子越大，p越小，r小于p的概率也越小，counter增长的概率也就越小。增长情况如下图：

![img](https://pic3.zhimg.com/80/v2-7c01512b9dd22cea15be4c051c31a46a_720w.jpg)

从左到右表示key的命中次数，从上到下表示影响因子，在影响因子为100的条件下，经过10M次命中才能把后8位值加满到255.

### 4.3、新生KEY策略

另外一个问题是，当创建新对象的时候，对象的counter如果为0，很容易就会被淘汰掉，还需要为新生key设置一个初始counter。counter会被初始化为LFU_INIT_VAL，默认5。