```properties
# 这个是生产搜集打点数据用的
# 0，定义组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

# 1，配置source
a1.sources.r1.type=org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.zookeeperConnect=nebula03:2181,nebula04:2181,nebula05:2181
a1.sources.r1.topic=pageview,event,timing,transaction,item

a1.sources.r1.groupId=flume_consumer
a1.sources.r1.kafka.consumer.timeout.ms=100
a1.sources.r1.batchSize=1000
a1.sources.r1.batchDurationMillis=1000
a1.sources.r1.channels=c1

# 2，配置拦截器
a1.sources.r1.interceptors=i1 i2
a1.sources.r1.interceptors.i1.type=com.lebbay.flume.interceptor.IMTopicInterceptor$Builder
a1.sources.r1.interceptors.i2.type=com.lebbay.flume.interceptor.IMTimeStampInterceptor$Builder

# 3, 定义channel相关的信息
a1.channels.c1.type=memory
a1.channels.c1.capacity=1000000
a1.channels.c1.transactionCapacity=10000
a1.channels.c1.keep-alive = 60

# 4, 定义sink相关的信息
a1.sinks.k1.type=hdfs
a1.sinks.k1.hdfs.path=/flume/%{topic}/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix=%{topic}
a1.sinks.k1.hdfs.idleTimeout=300
a1.sinks.k1.hdfs.rollSize=134217728
a1.sinks.k1.hdfs.rollCount=200000  
a1.sinks.k1.hdfs.rollInterval=3600
a1.sinks.k1.hdfs.threadsPoolSize=10
a1.sinks.k1.hdfs.fileType=DataStream    
a1.sinks.k1.hdfs.writeFormat=Text
a1.sinks.k1.channel=c1
# a1.sinks.k1.hdfs.round = false
# a1.sinks.k1.hdfs.roundValue = 30
# a1.sinks.k1.hdfs.roundUnit = second
```

