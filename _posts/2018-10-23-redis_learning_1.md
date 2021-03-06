---
layout: blog
title: '【Redis学习笔记一】Redis特点、常用命令'
date: 2018-10-24 12:11:34
categories: Redis-Learning
tags: code
lead_text: '介绍Redis特点、常用命令、数据结构'
---


### Redis的特性

- 速度快
- 持久化（断电不丢数据）
- 多种数据结构
- 支持多种客户端语言
- 功能丰富
- 操作简单
- 主从复制
- 高可用，分布式

### Redis的通用命令
#### keys：计算键

``` shell
key *  # 遍历所有key
```

keys命令支持正则匹配，如`keys h*`表示便利店以h开头的所有key。
因为redis是单线程，keys命令一般不在生产环境中使用。

keys *怎么用？

- 热备从节点
- scan

#### dbsize：计算key的总数

  `dbsize`

  时间复杂度是O(1)，可以在线上随意使用

#### exists：判断key是否存在

  `exists key`

  时间复杂度是O(1)，存在返回1，不存在返回0

#### del：删除指定的key，可以同时删除多个

  `del key`

  成功删除返回1，如果key不存在返回0

#### expire：给key设置过期时间

  `expire key seconds`

#### ttl：查看剩余过期时间

  `tth key`

  返回-2表明key已经不存在，被删除了；返回-1表明key存在，但没有设置过期时间

#### persist：撤销过期时间

  `persist key`

#### type：返回key的类型

  `type key`

#### 时间复杂度

  | 命令   | 时间复杂度 |
  | :------:| :-----: |
  | keys   | O(n)       |
  | dbsize | O(1)       |
  | del    | O(1)       |
  | exists | O(1)       |
  | expire | O(1)       |
  | type   | O(1)       |


### 数据结构

#### string
**结构和命令**

  - 可以是“字符串”、“数字”、“二进制”
  - value limited：Up to 512MB

**使用场景**

  - 缓存
  - 计数器
  - 分布式锁

**常用命令**

`get key`

  - `mget key1 key2 ...`：批量获取key，原子操作
  - `mset key1 value1 key2 value2 ...`：批量设置key-value

`set key`

  - `set key value`：不管key是否存在，都设置
  - `setnx key value`：key不存在时才设置
  - `set key value xx`：key存在时才设置

`del key`

`incr`：自增1

`decr`：与`incr`相反

`incrby`：自增k

`decrby`：与`incrby`相反

- 查缺补漏

  - `getset key newvalue`：设置新的value并返回旧的value，O(1)
  - `append key value`：将value追加到旧的value，O(1)
  - `strlen key`：返回字符串的长度（注意中文，每个中文占2个字节），O(1)
  - `incrbyfloat key 1.1`：自增浮点数，自减时设置负值就ok
  - `getrange key start end `：获取字符串指定下标的所有值
  - `setrange key index value`：设置指定下标多有对应的值

  | 命令  | 复杂度 |
  | :----: | :----: |
  | `set key value`    | O(1)   |
  | `get key`          | O(1)   |
  | `del key`          | O(1)   |
  | `setnx` or `setxx` | O(1)   |
  | `incr` or `decr`   | O(1)   |
  | `mget` or `mset`   | O(n)   |

#### hash

  - 特点
    - 键值结构：key   field   value
  - 常用命令
    - `hget key field`：获取hash key对应field的value，O(1)
    - `hset key field`：设置hash key对应field的value，O(1)
    - `hdel key field`：删除hash key对应field的value，O(1)
    - `hexists key field`：判断hash key是否有field，O(1)
    - `hlen key`：获取hash key field的数量
    - `hmget key field1 field2 ...`：批量获取hash key的一批field对应的值，O(n)
    - `hmset key1 field1 key2 field2...`：批量设置hash key的一批field value，O(n)
  - 查缺补漏
    - `hsetnx key field value`：设置hash key对应field的value，如果field已经存在，则失败，O(1)
    - `hincrby key field intCounter`：hash key对应的field的value自增intCounter，O(1)
    - `hincrbyfloat key field floatCountry`：hincrby浮点数版，O(1)

#### list

  - 特点

    - 有序
    - 可重复
    - 左右两边插入弹出

  - 常用命令

    - `rpush key value1 value2...`：从列表右边插入值

    - `lpush key value1 value2...`：从列表左边插入值

    - `linsert key before|after value newValue`：在列表指定的值前or后插入值

    - `lpop key`：从左边弹出元素

    - `rpop key`：从右边弹出元素

    - `lrem key count value` ：根据count的值，从列表中删除所有value相等的项，O(n)

      （1）count>0，从左到右，删除最多count个value相等的项

      （2）count<0，从右到左，删除最多count个value相等的项

      （3）count=0，删除所有value相等的项

    - `ltrim key start end`：按照索引范围修建列表

    - `lrange key start end`：获取列表指定索引范围所有item，包含end

    - `lindex key index`：获取列表索引对应的value

    - `llen key`：获取列表长度

    - `lset key index newValue`：设置列表指定索引值为newValue

  - 查缺补漏

    - `blpop key timeout`：lpop 阻塞版本，timeout是阻塞超时时间，timeout=0为永远不阻塞，O(1)
    - `brpop key timeout`：rpop阻塞版本，timeout是阻塞超时时间，timeout=0为永远不阻塞，O(1)

  - Tips

    - `LPUSH + LPOP = Stack`
    - `LPUSH + RPOP = Queue`
    - `LPUSH + LTRIM = Capped Collection`
    - `LPUSH + BRPOP = Message Queue`

#### set

  - 特点
    - 无序
    - 不重复
  - 常用命令
    - `sadd key element`：向集合key添加element，如果element已经存在，则添加失败，O(1)
    - `srem key element`：将集合key中的element移除掉，O(1)
    - `scard user:1:follow `：计算集合大小
    - `sismember user:1:follow it `：判断it是否在集合中
    - `srandmember user:1:follow count `：从集合中随机挑count个元素，不会破坏集合
    - `smembers`：取出集合中的所有元素，返回结果是无序的，小心使用，容易阻塞redis
    - `spop user:1:follow `：从集合中随机弹出一个元素，会破坏集合
    - `sdiff user:1:follow user:2:follow`：求两个集合的差集
    - `sinter user:1:follow user:2:follow`：求两个集合的交集
    - `sunion user:1:follow user:2:follow`：求两个集合的并集
    - `sdiff|sinter|sunion + store destkey...`：将差集、交集、并集的结果保存在destkey中

#### zset

  - 特点
    - 无重复元素
    - 有序
    - score + element
  - 常用命令
    - `zadd key score element`：添加score和element，可以是多对，O(logN)
    - `zrem key element`：删除元素，O(1)
    - `zscore key element`：获取score，O(1)
    - `zincrby key increScore element`：增加或减少元素的score
    - `zcard key`：返回元素总个数，O(1)
    - `zrange key start end [WITHSCORES]`：返回指定索引范围内的升序元素[分值]，O(logN+m)
    - `zrangebyscore key minScore maxScore [WITHSCORES]`：返回指定分数范围内的升序元素[分值]，O(logN+m)
    - `zcount key minScore maxScore`：返回有序集合内在指定分数范围内的个数，O(logN+m)
    - `zremrangebyrank key start end`：删除置顶排名的升序元素，O(logN+m)
    - `zremrangebyscore key minScore maxScore`：删除置顶分数内的升序元素，O(logN+m)
  - 查缺补漏
    - `zrevrank`：zrank的倒序
    - `zrevrange`：zrange的倒序
    - `zrevrangebyscore`：zrangebyscore的倒序
    - `zinterstore`：有序集合的交集
    - `zunionstore`：有序集合的并集

- 记录网站访问量

  `incr userid:pageview`

  `hincrby user:1:info pageview count`

### 单线程架构

redis是单线程架构，命令是串行的，上一个命令执行完了才会执行下一条命令。

- 单线程为什么这么快？

  1.纯内存：内存的响应速度快（主要原因）

  2.非阻塞IO

  3.避免前程切换和竟态消耗

#### 单线程

- 拒绝长命令，如：keys，flushall，flushdb，slow lua script，multi/exec，operate big value
- redis其实不是完全单线程
  - fysnc file descriptor
  - close file descriptor





