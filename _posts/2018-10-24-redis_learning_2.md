---
layout: blog
title: '【Redis学习笔记二】Redis客户端'
date: 2018-10-24 12:11:34
categories: blog
tags: code
lead_text: '介绍Jedis和redis-py'
---


### Java客户端：Jedis

#### Jedis基本使用

##### string

>  jedis.set("hello", "world");
>
>  //[Out]: OK
>
>  jedis.get("hello");
>
>  //[Out]: world
>
>  jedis.incr("counter");
>
>  //[Out]: 1


##### hash

```java
jedis.hset("myhash", "f1", "v1");
jedis.hset("myhash", "f2", "v2");
jedis.hgetAll("myhash");
//[Out]: {f1=v1,f2=v2}
```

##### list

```java
jedis.rpush("mylist", "1");
jedis.rpush("mylist", "2");
jedis.rpush("mylist", "3");
jedis.lrange("mylist", 0, -1);
//[Out]: [1,2,3]
```

##### set

```java
jedis.sadd("myset", "a");
jedis.sadd("myset", "b");
jedis.sadd("myset", "a");
jedis.smember("myset");
//[Out]: [b,a]
```

##### zset

```java
jedis.zadd("myzset", 99, "tom");
jedis.zadd("myzset", 66, "paper");
jedis.zadd("myzset", 33, "james");
jedis.zrangeWithScore("myzset", 0, -1);
//[Out]: [[["james"], 33.0],[["paper"], 66.0],[["tom"], 99.0]]
```



### Python客户端：redis-py

#### 安装redis-py

```python
sudo pip install redis

easy_install redis
```

#### 简单使用

```python
import redis

client = redis.StrictRedis(host = "127.0.0.1", port = 6379)
key = "hello"
setResult = client.set(key, "python-redis")
print setResult
value = client.get(key)
print "key: " + key + ", value: " + value
```

##### string

```python
client.set("hello", "world")
# True
client.get("hello")
# world
client.incr("counter")
# 1
```

##### hash

```python
client.hset("myhash", "f1", "v1")
client.hset("myhash", "f2", "v2")
client.hgetall("myhash")
# {"f1":"v1", "f2":"v2"}
```

##### list

```python
client.rpush("mylist", "1")
client.rpush("mylist", "2")
client.rpush("mylist", "3")
client.lrange("mylist", 0, -1)
# ["1","2","3"]
```

##### set

```python
client.sadd("myset", "a")
client.sadd("myset", "b")
client.sadd("myset", "a")
client.smenber("myset")
# set(["a","b"])
```

##### zset

```python
client.zadd("myzset", "99", "tom")
client.zadd("myzset", "66", "pater")
client.zadd("myzset", "33", "james")
client.zrange("myzset", 0, -1, withscores=True)
# [("james",33.0),("pater",66.0),("tom",99.0)]
```



