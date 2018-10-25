---
layout: blog
title: '【PySpark学习笔记一】弹性分布式数据集'
date: 2018-10-25 12:11:34
categories: blog
tags: code
lead_text: '介绍弹性分布式数据集RDD'
---



### RDD的内部运行方式

Spark优势：每个转换操作并行执行，大大提高速度。

数据集的转换通常是惰性的，即在transformation过程不会执行程序，只有在action过程才会执行。

### 创建RDD

导入相关程序库

``` python
from pyspark import SparkContext as sc
from pyspark import SparkConf
```

创建RDD

``` python
# 将list或array转为RDD
data = sc.parallelize(
[("A", 2), ("B", 3), ("C", 4)])
# 从本地或外部文件导入
data_from_file = sc.textFile(path, n) # n为分区

```

#### Schema

RDD是无Schema的数据结构，因此可以混合使用tuple、dict、list等多种数据结构。

#### 全局作用域与局部作用域

Spark可以在两种模式下运行：本地模式和集群模式。

### transformation操作

#### .map(...)

该方法应用在每一个RDD元素上，与python中的`map`函数类似：

``` python
data_new = data.map(lambda row: (row,1))
```

#### .filter(...)

该方法对RDD中元素按照条件进行过滤：

``` python
data_filtered = data.filter(lambda row: row[1] == 2 or row[1] == 3)
```

#### .flatMap(...)

`.flatMap(…)`方法和`.map(...)`方法类似，但`.flatMap(…)`方法返回一个扁平结果，而不是一个列表。

``` python
data_flat = data.flatMap(lambda row: row[1]+1)
```

#### .distinct(...)

该方法返回指定列中不同值的列表：

``` python
data_distinct = data.map(lambda row: (row,1)).distinct()
```

#### .sample(...)

该方法返回数据集的随机样本，第一个参数指定采样是否应该替换，第二个参数定义返回数据的分数，第三个参数是伪随机数产生器的种子。

``` python
fraction = 0.1
data_sample = data.sample(False, fraction, 666)
```

上例中，从原数据集中随机抽样10%。

#### .leftOuterJoin(...)

与SQL中`left join`的语法类似，用以链接两个RDD：

``` python
rdd1 = sc.parallelize([('a', 1), ('b', 4), ('c', 10)])
rdd2 = sc.parallelize([('a', 4), ('a', 1), ('b', '6'), ('d', 11)])
rdd3 = rdd1.leftOuterJoin(rdd2)
```

运行结果如下：

``` python
rdd3.collect()
# Out[1]: [('c',(10, None)), ('b', (4,'6')), ('a', (1,4)), ('a', (1,1))]
```

此外，还有`.join(...)`和`.intersection(...)`方法，均与SQL中意义相同。

#### .repartition(...)

该方法重新对数据集进行分区，改变数据集分区的数量：

``` python
rdd1 = rdd1.repartition(4)
```

上例中，将`rdd1`重新设置了4个分区。

### Action操作

#### .take(...)

该方法用于返回RDD中单个分区的前n行元素：

``` python
rdd1.take(1)
```

如果想要随机的记录，可以使用`takeSample(...)`方法，这个方法有三个参数：第一个参数表示采样是否应该被替换，第二个参数指定要返回的记录数量，第三个参数是伪随机数发生器种子。

``` python
data_take_sample = data.takeSample(False, 1, 667)
```

#### .collect(...)

与`.take(...)`方法类似，但`.collect(...)`方法返回全部RDD元素，因此在大规模RDD时不推荐使用。

#### .reduce(...)

`.reduce(...)`方法使用指定的方法减少RDD中的元素。

``` python
rdd1.map(lambda row: row[1]).reduce(lambda x,y: x+y)
```

上述方法用于计算RDD总的元素数量，首先通过`.map(...)`转换，创建一个包含rdd1所有值的列表，然后使用`.reduce(...)`方法对结果进行处理。

`.reduceByKey(...)`方法和`.reduce(...)`方法类似，但`.reduceByKey(...)`是在键-键基础上进行：

``` python 
data_key = sc.parallelize([('a',4),('b',3),('c',2),('a',3),('d',2),('b',5),('d',1)], 4)
data_key.reduceByKey(lambda x, y: x + y).collect()
#Out[2]: [('b',8),('a',7),('c',2),('d',3)]
```

#### .count(...)

该方法用于统计RDD中元素数量，如果数据集是k-v格式的，可以使用`countByKey(...)`获取不同键的计数：

``` python
data.count()
data.countByKey().items()
```

#### .saveAsTextFile(...)

该方法将RDD保存为文件，每个文件一个分区：

``` python
data.saveAsTextFile(path + filename)
```

