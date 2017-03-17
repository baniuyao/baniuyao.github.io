---
title: Run pyspark in Python3
date: 2016-08-05 17:47:53
tags:
- spark
- sparksql
- hive
- python3
---

# Backgroud

We were using spark in Scala environment for a long time and it did its job well. Now we need to query spark in python3 scripts and this post is a simple how-to guide for pyspark.

# Prerequisites

1. jdk
2. spark
3. python3
4. set `JAVA_HOME` and `SPARK_HOME` in env

`pyspark` is in your `SPARK_HOME/python`, you need to put it into you python path. You can `cp` or `ln` the directory to you python3 path, another method is use `sys.path.append` in your scripts.

# Example

``` python
from pyspark import SparkContext, SparkConf
from pyspark.sql import HiveContext


if __name__ == '__main__':
    conf = SparkConf()
    conf.set('spark.executor.memory', '16g')
    sc = SparkContext('spark://10.3.99.42:7077', appName='test', conf=conf)
    sc.setLogLevel('ERROR')
    hiveContext = HiveContext(sc)
    hiveContext.setConf('hive.metastore.uris', 'thrift://127.0.0.1:9083')
    hiveContext.setConf('io.compression.codecs', 'org.apache.hadoop.io.compress.SnappyCodec')
    hiveContext.sql('use default')
    hiveContext.sql("select * from table").show()
```