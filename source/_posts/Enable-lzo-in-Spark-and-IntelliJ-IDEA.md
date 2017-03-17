---
title: Enable lzo in Spark and IntelliJ IDEA
date: 2016-07-08 09:32:13
tags:
- spark
- Intellij
- lzo
- hadoop
---

# Intro

LZO is a compression codec for Hadoop-environment component like hadoop, hbase, spark and so on. The problem I met is that I want to read and calculate data from Hive table in lzo by SparkSQL. In this post, I will show you how to enable lzo in a Spark box without extra hadoop lib and run SparkSQL in Intellij IDEA in Mac.

# Box 

* spark-1.6.2-bin-hadoop2.6
* jdk-1.7
* hadoop-lzo-0.4.20
* maven3

Spark node run on CentOS 7, and Intellij IDEA is on OS X EI Capitan.

# Enable lzo in spark node

You can run spark in standalone without any other component, so it is easy to test wheather we have enable lzo. 

#### Install linux lzo dependencies by yum

```
yum install -y lzo-devel lzop
```

#### clone hadoop-lzo repo from github

Clone it at [https://github.com/twitter/hadoop-lzo](https://github.com/twitter/hadoop-lzo). Then use maven.

``` 
git clone https://github.com/twitter/hadoop-lzo
cd hadoop-lzo
mvn package
```

After package it, you will get a jar named `hadoop-lzo-0.4.20-SNAPSHOT.jar` in `target` directory.

#### Add jar to spark env by set env in `${SPARK_HOME}/conf/spark-conf.sh`.

```
export SPARK_CLASSPATH=${HADOOP-LZO_HOME}/target/hadoop-lzo-0.4.20-SNAPSHOT.jar
```
	
`${SPARK_HOME}` and `${HADOOP-LZO_HOME}` is the directory you put spark and hadoop-lzo repo.
	
#### Run spark in standalone mode.

```
cd ${SPARK_HOME}
./sbin/start-master.sh
./sbin/start-slave.sh spark://${SPARK_IP}:${SPARK_PORT}
```
	
#### Test if lzo is availabe in spark by `spark-shell`.

```
cd ${SPARK_HOME}
./bin/spark-shell
scala> sqlContext.setConf("hive.metastore.uris", "thrift://cloudera-server:9083")

scala> sqlContext.setConf("io.compression.codecs", "com.hadoop.compression.lzo.LzoCodec,com.hadoop.compression.lzo.LzopCodec")

scala> sqlContext.sql("use default")
res2: org.apache.spark.sql.DataFrame = [result: string]

scala> sqlContext.sql("select cdn_ip, timestamp from total_cdn_log where date='2016-07-05' and hour='01' limit 10").show()
+-----------+----------+
|     cdn_ip| timestamp|
+-----------+----------+
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
|cdn_ip_none|1467651600|
+-----------+----------+
```
	
We must set `io.compression.codecs` to `com.hadoop.compression.lzo.LzoCodec,com.hadoop.compression.lzo.LzopCodec` because spark use snappy codec in default.
	
# Enable lzo in IntelliJ IDEA

I think you have make your SparkSQL job running - that is to say, scala box is okay in your IntelliJ IDEA. Mac OS is a little different with RedHat because it is based on FreeBSD and NetBSD's implementions.

#### Install lzo dependencies by `brew`
	
`brew` is an excellent package management tool in Mac which its full name is `Homebrew`. If you have not installed `brew`, you can go to its website [http://brew.sh/](http://brew.sh/). Its install is easy with only one line command.

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Okay, let's install `lzo` dependencies.

```
brew install lzo lzop
```

#### Package `hadoop-lzo`

**REMARK:** Please **DO NOT** copy the `hadoop-lzo` from other system - such as CentOS that we run spark on. Because the maven procedure in `hadoop-lzo` contains some `gcc` process which is strongly connected with the system.

To build `hadoop-lzo`, we need to specify the location we install `lzo`. 

```
$ brew list lzo
/usr/local/Cellar/lzo/2.09/include/lzo/ (13 files)
/usr/local/Cellar/lzo/2.09/lib/liblzo2.2.dylib
/usr/local/Cellar/lzo/2.09/lib/ (2 other files)
/usr/local/Cellar/lzo/2.09/share/doc/ (7 files)
```

Then we can get `hadoop-lzo` and package jar.

```
$ git clone https://github.com/twitter/hadoop-lzo
$ cd hadoop-lzo
$ C_INCLUDE_PATH=/usr/local/Cellar/lzo/2.09/include/ LIBRARY_PATH=/usr/local/Cellar/lzo/2.09/lib/   mvn clean package
```
**REMARK:** The guide [https://gist.github.com/zedar/c43cbc7ff7f98abee885](https://gist.github.com/zedar/c43cbc7ff7f98abee885) - Add LZO compresssion codecs to the Apache Hadoop and Spark is little wrong for set `C_INCLUDE_PATH`. In the guide, he said 
`C_INCLUDE_PATH=/usr/local/Cellar/lzo/2.06/include/lzo/`, it should be `C_INCLUDE_PATH=/usr/local/Cellar/lzo/2.09/include/`. We do not need add lzo suffix.

#### Config the IntelliJ IDEA box

The first step is add `hadoop-lzo-0.4.20-SNAPSHOT.jar` to your scala project.

![Alt text](/images/idea_lzo_add_jar.png)

The next step is add `java.library.path` to your jvm options in your menu `Run-Edit configurations...`. The option you need to add is like `-Djava.library.path=/Users/yaorenjie/Documents/git/hadoop-lzo/target`

![Alt text](/images/idea_lzo_add_jvm.png)

**REMARK:** You must specify the build directory you run maven. If you want to use other directory, you **MUST** move all files in `target`. The `hadoop-lzo` need native `gpl` library in `target` directory.

```
$ll native/Mac_OS_X-x86_64-64/lib
total 208
-rwxr-xr-x  1 yaorenjie  staff  21780 Jul  7 17:36 libgplcompression.0.dylib
-rw-r--r--  1 yaorenjie  staff   1164 Jul  7 17:36 libgplcompression.la
lrwxr-xr-x  1 yaorenjie  staff     25 Jul  7 17:36 libgplcompression.dylib -> libgplcompression.0.dylib
-rw-r--r--  1 yaorenjie  staff  69744 Jul  7 17:36 libgplcompression.a
```
If you want to dig it out, you can get this process in `GPLNativeCodeLoader`