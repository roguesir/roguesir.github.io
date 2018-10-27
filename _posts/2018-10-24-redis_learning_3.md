---
layout: blog
title: '【Redis学习笔记三】慢查询、pipeline、发布订阅、Bitmap、HyperLogLog、GEO'
date: 2018-10-24 12:11:34
categories: Redis-Learning
tags: code
lead_text: '介绍Redis的慢查询、pipeline、发布订阅、Bitmap、HyperLogLog、GEO'
---

- 目录
  - 慢查询
  - pipline
  - 发布订阅
  - Bitmap
  - HyperLogLog
  - GEO

## 慢查询

### 生命周期

发送命令 --> 排队 --> 执行命令 --> 返回结果

说明：

1.慢查询发生在第三阶段

2.客户端超时不一定有慢查询，四个阶段都可能会是超时的原因

### 两个配置

#### slowlog-max-len

1.先进先出队列

2.固定长度

3.保存在内存内，不会持久化

#### slowlog-log-slower-than

1.慢查询阈值（单位：微秒）

2.slowlog-log-slower-than=0，记录所有命令

3.slowlog-log-slower-than<0，不记录所有命令

**配置方法**

1.默认值

config get slowlog-max-len = 128

config get slowlog-log-slower-than = 10000

2.修改配置文件重启

3.动态配置

config get slowlog-max-len 100

config get slowlog-log-slower-than 1000

### 三个命令

``` shell
slowlog get [n] #获取慢查询队列
slowlog len #获取慢查询队列长度
slowlog reset #清空慢查询队列
```

### 运维经验

1.slowlog-max-len阈值不要设置过大，默人10ms，通常设置1ms

2.slowlog-log-slower-than不要设置过小，通常设置1000左右

3.理解命令的生命周期

4.定期持久化慢查询（针对第2条）



## pipeline

### 什么是pipeline

client -- 传输命令 -- 计算 -- 返回结果

一次时间 = 一次网络时间 + 一次命令时间

n次时间 = n次网络时间 + n次命令时间

**pipeline**：将命令进行批量打包，在server中计算n次，一次返回结果



### pipeline与原生操作对比

原生命令：原子的

pipeline：非原子的

### 使用建议

1.注意每次pipeline携带的数据量

2.pipeline每次只能作用在一个Redis节点上

3.M操作与pipeline的区别



## 发布订阅

### 角色

发布者（publisher）

订阅者（subscriber）

频道（channel）

### API

#### publish

发布命令：`publish channel message`，如：

``` shell
redis> publish sohu:tv "hello world"
(integer) 3 #订阅者个数
```

#### subscribe

订阅：`subscribe [channel] #一个或多个`，如：

``` shell
redis> subscribe sohu:tv
1) "subscribe"
2) "sohu:tv"
3) (integer) 1
1) "message"
2) "sohu:tv"
3) "hello world"
```

#### unsubscribe

取消订阅：`unsubscribe [channel] #一个或多个`，如：

``` shell
redis> unsubscribe sohu:tv
1) "unsubscribe"
2) "sohu:tv"
3) (integer) 0
```

#### 其他API

`psubscribe [pattern...]`：订阅模式

`punsubscribe [pattern...]`：退订指定的模式

`pubsub channels`：列出至少有一个订阅者的频道

`pubsub numsub [channel...]`：列出给定频道的订阅者数量

`pubsub numpat`：列出被订阅模式的数量

#### 与消息队列的区别

- 发布订阅：消费者都会收到
- 消息队列：消费者“抢”消息

## Bitmap

### 位图

#### setbit

`setbit key offset value`：给位图指定索引设置值

#### getbit

`getbit key offset`：获取位图指定索引的值

#### bitcount

`bitcount key [start end]`：获取位图指定范围（start到end，单位为字节，如果不指定就是获取全部）位值为1的个数

#### bitop

`bitop op destkey key [key...]`：做多个bitmap的and（交集）、or（并集）、not（非）、xor（异或）操作并将结果保存在destkey中

#### bitpos

`bitpos key targetBit [start] [end]`：计算位图指定范围（start到end，单位为字节，如果不指定就是获取全部）第一个便宜量对应的值等于targetBit的位置

## HyperLogLog

1.基于HyperLogLog算法：极小空间实现独立数量统计

2.本质还是字符串

### API

`pfadd key element [element...]`：向hyperloglog添加元素

`pfcount key [key...]`：计算hyperloglog的独立总数

`pfmerge destkey sourcekey [courcekey...]`：合并多个hyperloglog

``` shell
redis> pfadd 2018_10_24:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-4"
(integer) 1
redis> pfcount 2018_10_24:unique:ids
(integer) 4
redis> pfadd 2018_10_24:unique:ids "uuid-1" "uuid-2" "uuid-3" "uuid-90"
(integer) 1
redis> pfcount 2018_10_24:unique:ids
(integer) 5
redis> pfadd 2018_10_25:unique:ids "uuid-4" "uuid-6" "uuid-7"
(integer) 1
redis> pfmerge 2018_10_24_25:unique:ids 2018_10_24:unique:ids 2018_10_25:unique:ids
OK
redis> pfcount 2018_10_24_25:unique:ids
(integer) 8

```

### 使用经验

1.是否能容忍错误？（错误率：0.81%）

2.是否需要单条数据？

## GEO

GEO：地理信息定位，存储经纬度，计算两地距离，范围计算等



### API

#### geoadd

`geoadd key longitude latitude member [longitude latitude member...]`：增加地理位置信息

``` shell
redis> geoadd cities:locations 116.28 39.55 beijing
(integer) 1
redis> geoadd cities:locations 117.12 39.08 tianjin 114.29 38.01 shijiazhuang 118.01 39.38 tangshan 115.29 38.51 baoding
```

#### geopos

`geopos key member [member ...]`：获取地理位置信息

``` shell
redis> geopos cities:locations tianjin
1) 1) "117.12000042200099501"
   2) "39.0800000535766543"
```

#### geodist

`geodist key member1 member2 [unit]`：获取两个地理位置的距离，其中unit的可选项：m（米）、km（千米）、mi（英里）、ft（尺）

``` shell
redis> geodist cities:locations tianjin beijing km
"89.2061"
```







