[toc]





## 1, flume消费kafka写入hdfs

* 注意修改配置文件中的agent为agent

```shell
# 0, 首先指定三种组件：源,通道,槽
agent.sources = source_from_kafka
agent.channels = mem_channel
agent.sinks = hdfs_sink

# 1，配置source， configure source
agent.sources.source_from_kafka.type = org.apache.flume.source.kafka.KafkaSource
agent.sources.source_from_kafka.zookeeperConnect = nebula03:2181,nebula04:2181,nebula05:2181
agent.sources.source_from_kafka.topic = appevent.dev
agent.sources.source_from_kafka.groupId = flume
agent.sources.source_from_kafka.kafka.consumer.timeout.ms = 100
agent.sources.source_from_kafka.channels = mem_channel


# 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
agent.channels.mem_channel.type = memory

# 3, 定义sink相关的信息
# specify the channel the sink should use  
agent.sinks.hdfs_sink.channel = mem_channel
agent.sinks.hdfs_sink.type = logger
# defind hdfs sink
agent.sinks.hdfs_sink.type = hdfs 
# set store hdfs path
agent.sinks.hdfs_sink.hdfs.path = /flume/events/%Y-%m-%d
# set file size to trigger roll
agent.sinks.hdfs_sink.hdfs.rollSize = 0  
agent.sinks.hdfs_sink.hdfs.rollCount = 0  
agent.sinks.hdfs_sink.hdfs.rollInterval = 3600  
agent.sinks.hdfs_sink.hdfs.threadsPoolSize = 30
agent.sinks.hdfs_sink.hdfs.fileType=DataStream    
agent.sinks.hdfs_sink.hdfs.writeFormat=Text
agent.sinks.hdfs_sink.hdfs.filePrefix = appevent
```

