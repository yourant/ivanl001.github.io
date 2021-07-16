[toc]

## 默认配置, agent=tier1

```shell
# Please paste flume.conf here. Example:

# Sources, channels, and sinks are defined per
# agent name, in this case 'tier1'.
tier1.sources  = source1
tier1.channels = channel1
tier1.sinks    = sink1

# For each source, channel, and sink, set
# standard properties.
tier1.sources.source1.type     = netcat
tier1.sources.source1.bind     = 127.0.0.1
tier1.sources.source1.port     = 9999
tier1.sources.source1.channels = channel1
tier1.channels.channel1.type   = memory
tier1.sinks.sink1.type         = logger
tier1.sinks.sink1.channel      = channel1

# Other properties are specific to each type of
# source, channel, or sink. In this case, we
# specify the capacity of the memory channel.
tier1.channels.channel1.capacity = 100

```





## 第一版配置,  agent=agent

```shell
# 0, 首先指定三种组件：源,通道,槽
agent.sources = source_from_kafka
agent.channels = mem_channel
agent.sinks = hdfs_sink

# 1，配置source， configure source
agent.sources.source_from_kafka.type = org.apache.flume.source.kafka.KafkaSource
agent.sources.source_from_kafka.zookeeperConnect = nebula03:2181,nebula04:2181,nebula05:2181
agent.sources.source_from_kafka.topic = appevent.test,moduleevent.test,pageevent.test
agent.sources.source_from_kafka.groupId = flume
agent.sources.source_from_kafka.kafka.consumer.timeout.ms = 100
agent.sources.source_from_kafka.channels = mem_channel


# 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
agent.channels.mem_channel.type = memory

# 3, 定义sink相关的信息
# specify the channel the sink should use  
agent.sinks.hdfs_sink.channel = mem_channel
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



## 第二版拦截器配置, agent=agent

* 不然清楚为啥报错拦截器返回空, 拦截器代码如下

```shell
# 0, 首先指定三种组件：源,通道,槽
agent.sources = source_from_kafka
agent.channels = c_pageview c_event c_timing c_transaction c_item
agent.sinks = s_pageview s_event s_timing s_transaction s_item

# 1，配置source， configure source
agent.sources.source_from_kafka.type = org.apache.flume.source.kafka.KafkaSource
agent.sources.source_from_kafka.zookeeperConnect = nebula03:2181,nebula04:2181,nebula05:2181
agent.sources.source_from_kafka.topic = pageview.test,event.test,timing.test,transaction.test,item.test
agent.sources.source_from_kafka.groupId = flume
agent.sources.source_from_kafka.kafka.consumer.timeout.ms = 100
agent.sources.source_from_kafka.channels = c_pageview c_event c_timing c_transaction c_item

# 2，配置拦截器，interceptor
agent.sources.source_from_kafka.interceptors = i1
agent.sources.source_from_kafka.interceptors.i1.type = com.lebbay.flume.interceptor.IMTopicInterceptor$Builder

# 3，通过拦截器代码，配置通道选择器，selector
agent.sources.source_from_kafka.selector.type = multiplexing
agent.sources.source_from_kafka.selector.header = topic
agent.sources.source_from_kafka.selector.mapping.pageview = c_pageview
agent.sources.source_from_kafka.selector.mapping.event = c_event
agent.sources.source_from_kafka.selector.mapping.timing = c_timing
agent.sources.source_from_kafka.selector.mapping.transaction = c_transaction
agent.sources.source_from_kafka.selector.mapping.item = c_item

# 2, 定义channel相关的信息, 这里不需要制定输出，而是让sink指定输入的
agent.channels.c_pageview.type = memory
agent.channels.c_event.type = memory
agent.channels.c_timing.type = memory
agent.channels.c_transaction.type = memory
agent.channels.c_item.type = memory

# 3, 定义sink相关的信息
# specify the channel the sink should use  
agent.sinks.s_pageview.channel = c_pageview
agent.sinks.s_pageview.type = logger
# defind hdfs sink
agent.sinks.s_pageview.type = hdfs 
# set store hdfs path
agent.sinks.s_pageview.hdfs.path = /flume/pageview/%Y-%m-%d
# set file size to trigger roll
agent.sinks.s_pageview.hdfs.rollSize = 0
agent.sinks.s_pageview.hdfs.rollCount = 0
agent.sinks.s_pageview.hdfs.rollInterval = 3600
agent.sinks.s_pageview.hdfs.threadsPoolSize = 30
agent.sinks.s_pageview.hdfs.fileType=DataStream  
agent.sinks.s_pageview.hdfs.writeFormat=Text
agent.sinks.s_pageview.hdfs.filePrefix = pageview

# specify the channel the sink should use  
agent.sinks.s_event.channel = c_event
agent.sinks.s_event.type = logger
# defind hdfs sink
agent.sinks.s_event.type = hdfs 
# set store hdfs path
agent.sinks.s_event.hdfs.path = /flume/event/%Y-%m-%d
# set file size to trigger roll
agent.sinks.s_event.hdfs.rollSize = 0
agent.sinks.s_event.hdfs.rollCount = 0
agent.sinks.s_event.hdfs.rollInterval = 3600
agent.sinks.s_event.hdfs.threadsPoolSize = 30
agent.sinks.s_event.hdfs.fileType=DataStream  
agent.sinks.s_event.hdfs.writeFormat=Text
agent.sinks.s_event.hdfs.filePrefix = event

# specify the channel the sink should use  
agent.sinks.s_timing.channel = c_timing
agent.sinks.s_timing.type = logger
# defind hdfs sink
agent.sinks.s_timing.type = hdfs 
# set store hdfs path
agent.sinks.s_timing.hdfs.path = /flume/timing/%Y-%m-%d
# set file size to trigger roll
agent.sinks.s_timing.hdfs.rollSize = 0
agent.sinks.s_timing.hdfs.rollCount = 0
agent.sinks.s_timing.hdfs.rollInterval = 3600
agent.sinks.s_timing.hdfs.threadsPoolSize = 30
agent.sinks.s_timing.hdfs.fileType=DataStream  
agent.sinks.s_timing.hdfs.writeFormat=Text
agent.sinks.s_timing.hdfs.filePrefix = timing

# specify the channel the sink should use  
agent.sinks.s_transaction.channel = c_transaction
agent.sinks.s_transaction.type = logger
# defind hdfs sink
agent.sinks.s_transaction.type = hdfs 
# set store hdfs path
agent.sinks.s_transaction.hdfs.path = /flume/transaction/%Y-%m-%d
# set file size to trigger roll
agent.sinks.s_transaction.hdfs.rollSize = 0
agent.sinks.s_transaction.hdfs.rollCount = 0
agent.sinks.s_transaction.hdfs.rollInterval = 3600
agent.sinks.s_transaction.hdfs.threadsPoolSize = 30
agent.sinks.s_transaction.hdfs.fileType=DataStream  
agent.sinks.s_transaction.hdfs.writeFormat=Text
agent.sinks.s_transaction.hdfs.filePrefix = transaction

# specify the channel the sink should use  
agent.sinks.s_item.channel = c_item
agent.sinks.s_item.type = logger
# defind hdfs sink
agent.sinks.s_item.type = hdfs 
# set store hdfs path
agent.sinks.s_item.hdfs.path = /flume/item/%Y-%m-%d
# set file size to trigger roll
agent.sinks.s_item.hdfs.rollSize = 0
agent.sinks.s_item.hdfs.rollCount = 0
agent.sinks.s_item.hdfs.rollInterval = 3600
agent.sinks.s_item.hdfs.threadsPoolSize = 30
agent.sinks.s_item.hdfs.fileType=DataStream  
agent.sinks.s_item.hdfs.writeFormat=Text
agent.sinks.s_item.hdfs.filePrefix = item
```

* 不太清楚有啥问题，但是就是报错！！

* 知道是啥问题了，下面这里返回了null...应该调用intercept(Event event)，然后返回list

  ```shell
  @Override
      public List<Event> intercept(List<Event> list) {
          return null;
      }
  ```

  

```java
package com.lebbay.flume.interceptor;

import com.alibaba.fastjson.JSONObject;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.nio.charset.Charset;
import java.util.List;
import java.util.Map;

/**
 * @author : ivanl001
 * #Date   : 2020-04-27 3:41 PM
 * #Desc   : 主题拦截器，不同的kafka主题写入到不同的hdfs目录
 **/
public class IMTopicInterceptor implements Interceptor {

    @Override
    public void initialize() {
    }

    @Override
    public Event intercept(Event event) {

        String body = new String(event.getBody());
        Map<String, String> headers = event.getHeaders();

        System.out.println("body:" + body);

        headers.put("topic", "pageview");

        JSONObject jsonObject = JSONObject.parseObject(body);
        String type = jsonObject.getString("t");

        if ("pageview".equals(type)){
            headers.put("topic", "pageview");
        } else if ("event".equals(type)){
            headers.put("topic", "event");
        } else if ("timing".equals(type)){
            headers.put("topic", "timing");
        } else if ("transaction".equals(type)){
            headers.put("topic", "transaction");
        } else if ("item".equals(type)){
            headers.put("topic", "item");
        }
        return event;
    }

    @Override
    public List<Event> intercept(List<Event> list) {
        return null;
    }

    @Override
    public void close() {
    }

    public static class Builder implements Interceptor.Builder{
        @Override
        public Interceptor build() {
            return new IMTopicInterceptor();
        }
        @Override
        public void configure(Context context) {
        }
    }
}
```



```shell
Builder class not found. Exception follows.
java.lang.ClassNotFoundException: com.lebbay.flume.interceptor.IMTopicInterceptor$Builder
	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:264)
	at org.apache.flume.interceptor.InterceptorBuilderFactory.newInstance(InterceptorBuilderFactory.java:48)
	at org.apache.flume.channel.ChannelProcessor.configureInterceptors(ChannelProcessor.java:111)
	at org.apache.flume.channel.ChannelProcessor.configure(ChannelProcessor.java:82)
	at org.apache.flume.conf.Configurables.configure(Configurables.java:41)
	at org.apache.flume.node.AbstractConfigurationProvider.loadSources(AbstractConfigurationProvider.java:348)
	at org.apache.flume.node.AbstractConfigurationProvider.getConfiguration(AbstractConfigurationProvider.java:101)
	at org.apache.flume.node.PollingPropertiesFileConfigurationProvider$FileWatcherRunnable.run(PollingPropertiesFileConfigurationProvider.java:141)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:180)
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:294)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```



```shell
/opt/cloudera/parcels/CDH/lib/flume-ng/lib/
```

