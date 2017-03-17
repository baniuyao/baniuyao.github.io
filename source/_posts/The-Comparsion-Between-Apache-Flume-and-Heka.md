---
title: The Comparsion Between Apache Flume and Heka
date: 2016-12-01 17:45:32
tags:
- Flume
- Heka
- Apache
---

# Environment

This test's main purpose is to choose the tool used to collect log on servers. In this test, we use Apache Flume and Heka to collect raw log data and send them into Kafka. The Kafka is in all default configuration. Linux is CentOS 6.4 with 12 cores CPU, 45GB memory and JDK 1.7.

## Benchmark Metrics

1. mps - Message Per Second: This data is fetched from JMX port of Kafka. We get the total message count from Kafka, and then get this count again. The delta value is the message count in 1 second - mps. We can get mps by command line:

```shell
java -jar /tmp/cmdline-jmxclient-0.10.3.jar - localhost:9999 kafka.server:name=MessagesInPerSec,type=BrokerTopicMetrics Count
```

2. channelSize (only for Flume): This metrics is the number of messages Flume cannot handle in a while. It can be a measurement to determin if the Flume is health. A health Flume instance should keep this value in a low number. We can get this value by Flume http service:

```shell
curl -s localhost:34545/metrics | python -m json.tool | grep ChannelSize
```

3. %cpu and %mem: We get these two values directly by linux `top` command:

```shell
flume_pid=`jps -m | grep App | cut -d' ' -f1`flume_cpu=`top -b -n1 -p $flume_pid | grep $flume_pid | awk '{print$9}'`flume_mem=`top -b -n1 -p $flume_pid | grep $flume_pid | awk '{print$10}'`
```

## Benchmark Items

We used a tool to write log lines into log files with different mps - 5K, 10K, 50K.

# Flume Benchmarks

## Flume Configuration

**JVM**

```shell
JAVA_OPTS="-Xms256m -Xmx256m -Dflume.monitoring.type=http -Dflume.monitoring.port=34545"
```

**Flume Configuration**


	agent.sources = fileSource
	agent.channels = diskChannel memChannel
	agent.sinks = kafkaSink
	agent.sources.fileSource.type = exec
	agent.sources.fileSource.command = tail -F /tmp/example.log
	agent.sources.fileSource.channels = memChannel
	agent.channels.memChannel.type = memory	agent.channels.memChannel.capacity = 1000000	agent.channels.memChannel.transactionCapacity = 10000
	agent.sinks.kafkaSink.type = org.apache.flume.sink.kafka.KafkaSink	agent.sinks.kafkaSink.topic = flume_test	agent.sinks.kafkaSink.brokerList = kafka:9092	agent.sinks.kafkaSink.batchSize = 200	agent.sinks.kafkaSink.channel = memChannel

## Flume Benchmark Results

### 5K mps

![Alt text](/images/flume_heka/flume_5k_mps.png)

![Alt text](/images/flume_heka/flume_5k_mem.png)

### 10K mps

![Alt text](/images/flume_heka/flume_10k_mps.png)

![Alt text](/images/flume_heka/flume_10k_mem.png)

### 50K mps

![Alt text](/images/flume_heka/flume_50k_mps.png)

![Alt text](/images/flume_heka/flume_50k_mem.png)

# Heka Benchmarks

## Heka Configuration

	[accesslogs]	type = "LogstreamerInput"	log_directory = "/tmp"	file_match = 'example\.log'
	[PayloadEncoder]	append_newlines = false
	[KafkaOutput]	type = "KafkaOutput"	message_matcher = "TRUE"	topic = "heka_test"	addrs = ["kafka:9092"]	encoder = "PayloadEncoder"	max_buffer_time = 1000	max_buffered_bytes = 1048576	required_acks = "NoResponse"

## Heka Benchmark Results

### 5K mps

![Alt text](/images/flume_heka/heka_5k_mps.png)

![Alt text](/images/flume_heka/heka_5k_mem.png)

### 10K mps

![Alt text](/images/flume_heka/heka_10k_mps.png)

![Alt text](/images/flume_heka/heka_10k_mem.png)

### 50K mps

![Alt text](/images/flume_heka/heka_50k_mps.png)

![Alt text](/images/flume_heka/heka_50k_mem.png)

# The Comparison

## About Performance

As the charts said, both Flume and Heka can handle huge amout of log in very high speed. We usually deploy them on servers which act as business usage. That means, it would not take much pressure (if we use them behind Apache Kafka - another usage type is sending date **TO** Kafka, they could work well). Considering the Linux system costs, Heka takes advantages on %MEM while Flume does on %CPU. Heka is written in Golang so that the memory usage is much lower than Flume which is Java made. For %CPU, Flume uses less cpu than Heka. Chars are below:

![Alt text](/images/flume_heka/total_mem.png)

![Alt text](/images/flume_heka/total_cpu.png)

## About Usability

### Features

Flume's configuration files are written in YAML while Heka's in TOML. We can find out that writing in TOML is much more simple than YAML as well as we can understand more easily. As for the architectures, Flume is easier. It contains three main components - Source, Channel and Sink. `Source` defines the date source; `Sink` defines the destination of the date; `Channel` is the buffer date pipeline between `Source` and `Sink`. In our exmple, log files is the `Source` and Kafka is the `Sink`. `Source` puts date into `Channel` while `Sink` takes date from `Channel`. The architecture is like below:

![Alt text](/images/flume_heka/flume_arch.png)

For Heka, its architecture is a little complex. Its `Inputs`, `Outputs` are like Flume's `Source` and `Sink`. Heka adds many feature such as `Splitter`, `Decoder`, `Filter` and `Encoder` in addition. These feature enable Heka to handle date directly in Heka itself, in the same time, make Heka a more powerful tool in comparison of Flume. The architecture is like below: 

![Alt text](/images/flume_heka/heka_arch)

### Monitor

Both Flume and Heka enables HTTP API to get metrics. 

For Flume, we can add JVM parameters `-Dflume.monitoring.type=http -Dflume.monitoring.port=34545` to enable the HTTP API. And the date is like below (by `curl -s localhost:34545/metrics | python -m json.tool`):
	
	{   		"CHANNEL.memChannel": {
   			"ChannelCapacity": "1000000",
   			"ChannelFillPercentage": "0.0",
   			"EventPutAttemptCount": "10",
   			"EventPutSuccessCount": "10",
   			"EventTakeAttemptCount": "16",
   			"EventTakeSuccessCount": "10",
   			"StartTime": "1458887648261",
   			"StopTime": "0",
   			"Type": "CHANNEL"
   			},
   			...
   	}

For Heka, we need to add this in to its configuration file, `ticker_interval` defines the interval Heka refresh the data:


	[DashboardOutput]	ticker_interval = 1

The result is like (by ``):

	"outputs": [{
		"Name": "KafkaOutput",
		"ProcessMessageCount": {
			"representation": "count",
			"value": 756698
		},
		"ProcessMessageDiscards": {
			"representation": "count",
			"value": 0
		},
		"ProcessMessageFailures": {
			"representation": "count",
			"value": 0 }
		},
		...
	}

### Problems

1. Flume cannot read log file continously after restart. Flume uses `tail` to get log lines from log files so Flume does not know which line it has read or which line it need to start to read after Flume restarts. Heka will keep offset in a meta file and can read the date continously after restarting. In this point, Heka is better.

2. Flume cannot update log files according to the configurations. For example, we defined we need to get log files named 'access_*.log', Flume will not get the files working only when we modify the configuration files. For Heka, Heka will monitor the directory and update the log files in some interval time which is set in configuration files. The parameter is `rescan_interval`:

>During logfile rotation, or if the logfile is not originally present on the system, this interval is how often the existence of the logfile will be checked for. The default of 5 seconds is usually fine. This interval is in milliseconds.

# Conclusions

Both Flume and Heka are good choice for our solutions. Heka is written Golang and we can start/stop it in seconds. For Flume, it always takes minutes to initialize or stop. And Heka can handle the circumstance of continoulsy reading while Flume can not. I think Heka would be a better choice.

But the bad news is that Heka is deprecated by mozilla in May of 2016. We can get the detail inforation [here](https://mail.mozilla.org/pipermail/heka/2016-May/001059.html). I thinks there are too many features in Heka and that makes Heka a heavy tool. I like it very much and my plan is to remove the additional components of Heka and make it a simple but powerful tool.


