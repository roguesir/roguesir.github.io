---
layout: blog
title: '【PySpark学习笔记三】spark-submit'
date: 2018-10-24 12:11:34
categories: blog
tags: code
lead_text: '介绍spark-submit'
---

spark-submit命令利用可重用的模块形式编写脚本，并且以编程方式提交作业到Spark。

### spark-submit命令

spark-submit命令提供一个统一的API把应用程序部署到各种Spark支持的集群管理器上，从而免除了单独配置每个应用程序。

#### 命令行参数

下面逐个介绍这些参数：

- `--master`：用于设置主结点URL的参数。
  - `local`：用于执行本地机器的代码。Spark运行一个单一的线程，在一个多核机器上，通过`local[n]`来指定一个具体使用的内核数，n指使用的内核数目，`local[*]`来指定运行和Spark机器内核一样多的复杂线程。
  - `spark://host:port`：这是一个URL和一个Spark单机集群的端口。
  - `mesos://host:port`：这是一个URL和一个部署在Mesos的Spark集群的端口。
  - `yarn`：作为负载均衡器，用于从运行Yarn的头结点提交作业。
- `--deploy-mode`：允许决定是否在本地（使用client）启动Spark驱动成簇的参数，或者在集群内（使用cluster选项）的其中一台工作机器上启动。默人是client。
- `--name`：应用程序名称。注意，创建SparkSession时，如果是以编程方式指定应用程序名称，那么来自命令行的参数会被重写。
- `--py-files`：`.py`、`.egg`或者`.zip`文件的逗号分隔列表，包括Python应用程序，这些文件将被交付给每一个执行器来使用。
- `--files`：命令给出一个逗号分隔的文件列表，这些文件将被交付给每一个执行器来使用。
- `--conf`：参数通过命令行动态地更改应用程序的配置。语法是：`<Spark property>=<value for the property>`。
- `--properties-file`：配置文件。它应该有和`conf/spark-defaults.conf`文件相同的属性设置，也是可读的。
- `--driver-memory`：指定应用程序在驱动程序上分配多少内存的参数。允许的值又一个语法限制，类似于1000M，2G。默认值是1024M。
- `--exectuor-memory`：参数指定每个执行器为应用程序分配多少内存。默认值是1G。
- `--help`：展示帮助信息和退出。
- `--verbose`：在运行应用程序时打印附加调试信息。
- `--version`：打印Spark版本。

仅在Spark单机集群（cluster）部署模式下，或者在一个Yarn上的部署集群上，可以使用`--driver-cores`来允许指定驱动程序的内核数量，默认值为1。尽在一个Spark单机或者Mesos集群（cluster）部署模型中，一下参数或许会用到：

- `--supervise`：当驱动程序丢失或者失败时，就会重新启动该驱动程序。
- `--kill`：将完成的过程赋予submission_id。
- `--status`：请求应用程序的状态。

在Spark单机和Mesos（client部署模式）中，可以指定`--total-exectuor-cores`，该参数会为所有执行器（不是每一个）请求指定的内核数量。另一方面，在Spark单机和YARN中，只有`--executor-cores`参数指定每个执行器的内核数量（在YARN模式中默认值为1，或者对于单机模式下所有工作节点可用的内核）。

另外，向YARN集群提交时你可以指定：

`--queue`：指定YARN上的队列，一边将改作业提交到队列（默认值是default）。

`--num-executors`：指定需要多少个执行器来请求改作业的参数。如果启动了动态分配，则执行器的初始数量至少是指定的数量。

### 创建SparkSession

``` python
from pyspark.sql import SparkSession

spark = SparkSession \
		.builder \
    	.appName("AppName") \
        .getOrCreate()

print "Session created!"
```

当SparkSession在后台启动时，不需要在创建一个SparkContext，为了获得访问权，可以简单调用`sc = spark.SparkContext`。





