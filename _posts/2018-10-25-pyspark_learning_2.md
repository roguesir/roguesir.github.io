---
layout: blog
title: '【PySpark学习笔记二】DataFrame'
date: 2018-10-25 12:11:34
categories: blog
tags: code
lead_text: '介绍DataFrame'
---

DataFrame是一种不可变的分布式数据集。Spark早期的API中，由于JVM和Py4J之间的通信开销，使用Python执行查询会明显变慢。

**Python到RDD之间的通信**

在PySpark驱动器中，Spark Context通过Py4J启动一个JavaSparkContext的JVM，所有的RDD转换最初都映射到Java中的PythonRDD对象。这样，Python和JVM之间就存在很多上下文切换和通信开销。

**利用DataFrame加速PySpark**

DataFrame和Catalyst优化器的意义在于和非优化的RDD查询比较时增加PySpark的性能，这种查询性能的提升源于降低了Python和JVM之间的通信开销。

### 创建DataFrame

通常情况下，使用SparkSession导入数据来创建DataFrame。

#### 生成JSON数据

``` python
stringJSONRDD = sc.parallelize(
("""{"id":"1","name":"a","age":"20"}""",
"""{"id":"2","name":"b","age":"21"}""",
"""{"id":"3","name":"c","age":"22"}""")
)
```

#### 创建一个DataFrame

``` python
peopleJSON = spark.read.json(stringJSONRDD)
```

#### 创建一个临时表

``` python
peopleJSON = createOrReplaceTempView("peopleJSON")
```

### 简单的DataFrame查询

#### DataFrame API查询

``` python
peopleJSON.show()
```

#### SQL查询

``` python
spark.sql("select * from peopleJSON").collect() # 查询表中全部数据
spark.sql("select * from peopleJSON").show(n) # 查询前n行数据
spark.sql("select * from peopleJSON").take(n) # 查询第n行数据
```

### RDD的交互操作

#### 使用反射来推断模式

在DataFrame中，键是列，数据类型通过采样数据来判断。

``` python
# 打印模式
peopleJSON.printSchema()
```

#### 编程指定模式

``` python
# 导入“类型”
from pyspark.sql.types import *

# 生成以逗号分隔的数据
stringCSVRDD = sc.parallelize(
    [
        (1,"a",20),
        (2,"b",21),
        (3,"c",22)
    ])

# 指定模式
schema = StructType(
    [
        StructField("id",LongType(),True),
        StructField("name",StringType(),True),
        StructField("age",LongType(),True),
    ])

# 创建DataFrame
people = spark.createDataFrame(stringCSVRDD, schema)

# 利用DataFrame创建临时视图
people.createOrReplaceTempView("people")
```

### 利用DataFrame API查询

#### 行数

可以通过collect、show、take等方法查看DataFrame中的数据。

``` python
people.count()
# [Out]: 3
```

#### 运行筛选语句

使用filter方法进行筛选：

``` python
people.select("id", "age").filter("age = 22").show()
# 和下面语句等效
people.select(people.id, people.age).filter(people.age == 22).show()
```

也可以使用like进行模式匹配：

``` python
people.select("name", "age").filter("name like 'a%'").show()
```

### 利用SQL查询

#### 行数

``` python
spark.sql("select count(1) from people").show()
```

#### 利用where语句进行条件筛选

``` python
spark.sql("select id, age from people where age = 22").show()

spark.sql("select name, age from people where name like 'a%'").show()
```













