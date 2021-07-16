[toc]

## 1, mysql连接

```shell
地址：report-system.cbb0nles4v8i.us-east-1.rds.amazonaws.com  
端口：3306
用户名：admin
密码：Jde27dePlWbx32aMTr


azaziedbslave.cbb0nles4v8i.us-east-1.rds.amazonaws.com
aztech1
8m4UquCBkNYn


# 在集群服务器上连接业务数据库-上面的数据库是报表数据库
# 上面两个连接都无法成功
ERROR 2003 (HY000): Can't connect to MySQL server on 'report-system.cbb0nles4v8i.us-east-1.rds.amazonaws.com' (110)
```



## 2, hdfs基准测试

### 01, hdfs的读写测试

#### 测试HDFS写性能

* 在cdh中：/opt/cloudera/parcels/CDH/jars/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.12.1-tests.jar

* 测试内容：向HDFS集群写10个128M的文件
* 测试完成后会发现hdfs的根目录下多了一些文件：在目录/benchmarks/TestDFSIO下面

```shell
# 原生
hadoop jar /opt/module/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.2-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB

# cdh中测试
# 先切换到hdfs用户上
su hdfs
hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.12.1-tests.jar TestDFSIO -write -nrFiles 10 -fileSize 128MB
```



写入性能如下：

```shell
20/04/10 23:31:35 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
20/04/10 23:31:35 INFO fs.TestDFSIO:            Date & time: Fri Apr 10 23:31:35 PDT 2020
20/04/10 23:31:35 INFO fs.TestDFSIO:        Number of files: 10
20/04/10 23:31:35 INFO fs.TestDFSIO: Total MBytes processed: 1280.0
20/04/10 23:31:35 INFO fs.TestDFSIO:      Throughput mb/sec: 95.50813311446053
20/04/10 23:31:35 INFO fs.TestDFSIO: Average IO rate mb/sec: 103.05743408203125
20/04/10 23:31:35 INFO fs.TestDFSIO:  IO rate std deviation: 26.913258508638148
20/04/10 23:31:35 INFO fs.TestDFSIO:     Test exec time sec: 17.964
20/04/10 23:31:35 INFO fs.TestDFSIO:
```

可以看到，不咋地哈，虚拟机

```java
19/08/03 00:58:25 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
19/08/03 00:58:25 INFO fs.TestDFSIO:            Date & time: Sat Aug 03 00:58:25 CST 2019
19/08/03 00:58:25 INFO fs.TestDFSIO:        Number of files: 10
19/08/03 00:58:25 INFO fs.TestDFSIO: Total MBytes processed: 1280.0
19/08/03 00:58:25 INFO fs.TestDFSIO:      Throughput mb/sec: 0.9841694801355386
19/08/03 00:58:25 INFO fs.TestDFSIO: Average IO rate mb/sec: 3.8536438941955566
19/08/03 00:58:25 INFO fs.TestDFSIO:  IO rate std deviation: 6.161262399312691
19/08/03 00:58:25 INFO fs.TestDFSIO:     Test exec time sec: 380.467
19/08/03 00:58:25 INFO fs.TestDFSIO: 
```

### 02, 测试HDFS的读性能

* 和写入测试用的是同一个jar包

```shell
# 先切换到hdfs用户上
su hdfs
# cdh中测试
hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.12.1-tests.jar TestDFSIO -read -nrFiles 10 -fileSize 128MB
```

读取性能如下：

```shell
20/04/10 23:34:05 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
20/04/10 23:34:05 INFO fs.TestDFSIO:            Date & time: Fri Apr 10 23:34:05 PDT 2020
20/04/10 23:34:05 INFO fs.TestDFSIO:        Number of files: 10
20/04/10 23:34:05 INFO fs.TestDFSIO: Total MBytes processed: 1280.0
20/04/10 23:34:05 INFO fs.TestDFSIO:      Throughput mb/sec: 644.8362720403022
20/04/10 23:34:05 INFO fs.TestDFSIO: Average IO rate mb/sec: 821.5789794921875
20/04/10 23:34:05 INFO fs.TestDFSIO:  IO rate std deviation: 421.22356647815394
20/04/10 23:34:05 INFO fs.TestDFSIO:     Test exec time sec: 15.984
20/04/10 23:34:05 INFO fs.TestDFSIO:
```



```java
19/08/03 01:04:09 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
19/08/03 01:04:09 INFO fs.TestDFSIO:            Date & time: Sat Aug 03 01:04:09 CST 2019
19/08/03 01:04:09 INFO fs.TestDFSIO:        Number of files: 10
19/08/03 01:04:09 INFO fs.TestDFSIO: Total MBytes processed: 1280.0
19/08/03 01:04:09 INFO fs.TestDFSIO:      Throughput mb/sec: 17.12282954758274
19/08/03 01:04:09 INFO fs.TestDFSIO: Average IO rate mb/sec: 82.3775634765625
19/08/03 01:04:09 INFO fs.TestDFSIO:  IO rate std deviation: 163.41886702019727
19/08/03 01:04:09 INFO fs.TestDFSIO:     Test exec time sec: 88.509
19/08/03 01:04:09 INFO fs.TestDFSIO: 
```



### 03, 删除刚才测试产生的数据

```shell
# 先切换到hdfs用户上
su hdfs
hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-mapreduce-client-jobclient-2.6.0-cdh5.12.1-tests.jar TestDFSIO -clean
```



### 04, 总结

```shell
20/04/10 23:31:35 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
20/04/10 23:31:35 INFO fs.TestDFSIO:            Date & time: Fri Apr 10 23:31:35 PDT 2020
20/04/10 23:31:35 INFO fs.TestDFSIO:      Throughput mb/sec: 95.50813311446053
20/04/10 23:31:35 INFO fs.TestDFSIO: Average IO rate mb/sec: 103.05743408203125
20/04/10 23:31:35 INFO fs.TestDFSIO:  IO rate std deviation: 26.913258508638148

20/04/10 23:34:05 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
20/04/10 23:34:05 INFO fs.TestDFSIO:            Date & time: Fri Apr 10 23:34:05 PDT 2020
20/04/10 23:34:05 INFO fs.TestDFSIO:      Throughput mb/sec: 644.8362720403022
20/04/10 23:34:05 INFO fs.TestDFSIO: Average IO rate mb/sec: 821.5789794921875
20/04/10 23:34:05 INFO fs.TestDFSIO:  IO rate std deviation: 421.22356647815394
```



## 3, MapReduce的性能测试

### 01, 生成数据

* 使用RandomWriter来产生随机数，每个节点运行10个Map任务，每个Map产生大约1G大小的二进制随机数

```shell
hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-mapreduce-examples-2.6.0-cdh5.12.1.jar randomwriter random-data
```

### 02, 执行Sort程序

```shell
hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-mapreduce-examples-2.6.0-cdh5.12.1.jar sort random-data sorted-data
```

### 03, 验证是否排好

```shell
# 这个好像不好使？
hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-mapreduce-examples-2.6.0-cdh5.12.1.jar testmapredsort -sortInput random-data -sortOutput sorted-data
```





## 4, kafka压力测试

### 01, 生产能力测试

```shell
# 生产能力测试命令如下：
/opt/cloudera/parcels/KAFKA/bin/kafka-producer-perf-test --topic test --record-size 100 --num-records 100000 --throughput 1000 --producer-props bootstrap.servers=nebula03:9092,nebula04:9092,nebula05:9092,nebula06:9092,nebula07:9092
```

说明：record-size是一条信息有多大，单位是字节。num-records是总共发送多少条信息。throughput 是每秒多少条信息。

结果如下：

```shell
20/04/11 01:02:58 INFO producer.KafkaProducer: [Producer clientId=producer-1] Closing the Kafka producer with timeoutMillis = 9223372036854775807 ms.
100000 records sent, 999.840026 records/sec (0.10 MB/sec), 1.41 ms avg latency, 346.00 ms max latency, 1 ms 50th, 2 ms 95th, 2 ms 99th, 42 ms 99.9th.
```



```java
19/08/03 19:12:08 INFO producer.KafkaProducer: [Producer clientId=producer-1] Closing the Kafka producer with timeoutMillis = 9223372036854775807 ms.
100000 records sent, 999.790044 records/sec (0.10 MB/sec), 2.72 ms avg latency, 475.00 ms max latency, 1 ms 50th, 2 ms 95th, 48 ms 99th, 381 ms 99.9th.

// 参数解析：本例中一共写入10w条消息，每秒向Kafka写入了0.10MB的数据，平均是1000条消息/秒，每次写入的平均延迟为2.72毫秒，最大的延迟为475毫秒。
```



### 02, 消费能力测试

```shell
# 消费能力测试如下
/opt/cloudera/parcels/KAFKA/bin/kafka-consumer-perf-test --broker-list  nebula03:9092,nebula04:9092,nebula05:9092,nebula06:9092,nebula07:9092 --topic test --fetch-size 10000 --messages 10000000 --threads 1

# --fetch-size 指定每次fetch的数据的大小
# --messages 总共要消费的消息个数

```



结果如下：

```shell
20/04/11 01:07:28 INFO internals.AbstractCoordinator: [Consumer clientId=consumer-1, groupId=perf-consumer-79436] Member consumer-1-db4eef92-2e73-4334-b3d0-3e39596f4e4e sending LeaveGroup request to coordinator nebula06:9092 (id: 2147483479 rack: null)
2020-04-11 01:07:14:512, 2020-04-11 01:07:28:860, 9.5367, 0.6647, 100000, 6969.6125, 3053, 11295, 0.8443, 8853.4750
```



```shell
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
2019-08-03 19:19:24:566, 2019-08-03 19:19:40:258, 9.5367, 0.6077, 100000, 6372.6740, 3123, 12569, 0.7588, 7956.0824

开始测试时间，测试结束时间，最大吞吐率9.5367MB/s，平均每秒
```



### 03, 总结

```shell
# 生产
20/04/11 01:02:58 INFO producer.KafkaProducer: [Producer clientId=producer-1] Closing the Kafka producer with timeoutMillis = 9223372036854775807 ms.
100000 records sent, 999.840026 records/sec (0.10 MB/sec), 1.41 ms avg latency, 346.00 ms max latency, 1 ms 50th, 2 ms 95th, 2 ms 99th, 42 ms 99.9th.
# 消费
20/04/11 01:07:28 INFO internals.AbstractCoordinator: [Consumer clientId=consumer-1, groupId=perf-consumer-79436] Member consumer-1-db4eef92-2e73-4334-b3d0-3e39596f4e4e sending LeaveGroup request to coordinator nebula06:9092 (id: 2147483479 rack: null)
2020-04-11 01:07:14:512, 2020-04-11 01:07:28:860, 9.5367, 0.6647, 100000, 6969.6125, 3053, 11295, 0.8443, 8853.4750
```



