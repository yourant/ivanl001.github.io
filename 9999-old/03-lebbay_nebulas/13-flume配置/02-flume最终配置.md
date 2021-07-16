*  测试配置

```shell
# 0，定义组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

# 1，配置source
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.zookeeperConnect = nebula03:2181,nebula04:2181,nebula05:2181
a1.sources.r1.topic = pageview.test,event.test,timing.test,transaction.test,item.test
a1.sources.r1.groupId = flume_consumer
a1.sources.r1.kafka.consumer.timeout.ms = 100
a1.sources.r1.batchSize = 1000
a1.sources.r1.batchDurationMillis = 1000
a1.sources.r1.channels = c1

# 2，配置拦截器
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.lebbay.flume.interceptor.IMTopicInterceptor$Builder

# 3, 定义channel相关的信息
a1.channels.c1.type=memory
a1.channels.c1.capacity=100000
a1.channels.c1.transactionCapacity=10000

# 4, 定义sink相关的信息
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /flume/%{topic}/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = %{topic}-
a1.sinks.k1.hdfs.rollSize = 0  
a1.sinks.k1.hdfs.rollCount = 0  
a1.sinks.k1.hdfs.rollInterval = 3600  
a1.sinks.k1.hdfs.threadsPoolSize = 30
a1.sinks.k1.hdfs.fileType=DataStream    
a1.sinks.k1.hdfs.writeFormat=Text
a1.sinks.k1.hdfs.filePrefix = appevent
a1.sinks.k1.channel= c1
# a1.sinks.k1.hdfs.round = false
# a1.sinks.k1.hdfs.roundValue = 30
# a1.sinks.k1.hdfs.roundUnit = second
```



* 拦截器代码

```java
package com.lebbay.flume.interceptor;

import com.alibaba.fastjson.JSONObject;
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.interceptor.Interceptor;

import java.util.ArrayList;
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
    public List<Event> intercept(List<Event> events) {
        for (Event event : events) {
            //传递的是引用，所以此处的events中的改变了原数据，所以不用接收返回值
            intercept(event);
        }
        return events;
    }

    @Override
    public void close() {
    }

    public static class Builder implements Interceptor.Builder {

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



* 发送拦截器jar包到flume的lib目录下
* 该jar包下依赖的jar包也需要放进去，如fastjson

```shell
/opt/cloudera/parcels/CDH/lib/flume-ng/lib/
```





```shell
# 0，定义组件
a1.sources=r1
a1.channels=c1
a1.sinks=k1

# 1，配置source
a1.sources.r1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.r1.zookeeperConnect = nebula03:2181,nebula04:2181,nebula05:2181
a1.sources.r1.topic = pageview.test,event.test,timing.test,transaction.test,item.test
a1.sources.r1.groupId = flume_consumer
a1.sources.r1.kafka.consumer.timeout.ms = 100
a1.sources.r1.batchSize = 1000
a1.sources.r1.batchDurationMillis = 1000
a1.sources.r1.channels = c1

# 2，配置拦截器
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.lebbay.flume.interceptor.IMTopicInterceptor$Builder

# 3, 定义channel相关的信息
a1.channels.c1.type=memory
a1.channels.c1.capacity=100000
a1.channels.c1.transactionCapacity=10000

# 4, 定义sink相关的信息
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = /flume/%{topic}-test/%Y-%m-%d
a1.sinks.k1.hdfs.filePrefix = %{topic}-test
a1.sinks.k1.hdfs.rollSize = 134217728
a1.sinks.k1.hdfs.rollCount = 0  
a1.sinks.k1.hdfs.rollInterval = 0  
a1.sinks.k1.hdfs.threadsPoolSize = 30
a1.sinks.k1.hdfs.fileType=DataStream    
a1.sinks.k1.hdfs.writeFormat=Text
a1.sinks.k1.channel= c1
# a1.sinks.k1.hdfs.round = false
# a1.sinks.k1.hdfs.roundValue = 30
# a1.sinks.k1.hdfs.roundUnit = second
```







