

```shell
/usr/local/filebeat/filebeat -e -c /usr/local/filebeat/filebeat.yml


kafka-console-consumer --bootstrap-server nebula06:9092 --topic nginx_log_test




im.ivanl001.A02_HotItemAnalysis_kafka



error ompiling the sbt component 'compiler-interface'
```







```java

package im.ivanl001

import java.sql.Timestamp
import java.util.Properties

import org.apache.flink.api.common.functions.AggregateFunction
import org.apache.flink.api.common.restartstrategy.RestartStrategies
import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.api.common.state.{ListState, ListStateDescriptor}
import org.apache.flink.api.java.tuple.{Tuple, Tuple1}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.environment.CheckpointConfig
import org.apache.flink.streaming.api.functions.KeyedProcessFunction
import org.apache.flink.streaming.api.scala.function.WindowFunction
import org.apache.flink.streaming.api.scala.{KeyedStream, StreamExecutionEnvironment, _}
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.streaming.api.windowing.windows.TimeWindow
import org.apache.flink.streaming.api.{CheckpointingMode, TimeCharacteristic}
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer010
import org.apache.flink.util.Collector

import scala.collection.mutable.ListBuffer

/**
  * #author      : ivanl001
  * #creator     : 2019-08-23 21:23
  * #description : 根据商品点击信息，实时统计热门商品信息
  *                使用窗口函数，过去一小时的点击信息，每5分钟窗口滚动一次，也就是统计一次。时间按照事件时间进行处理。
  *                在开启窗口后，使用aggregate进行窗口函数的预聚合和最后窗口信息聚合输出
  *                输出后，再根据window.getEnd的窗口结束时间进行keyby操作
  *                然后把keyby分组的数据进行process(new TopNHotItems01(5))，保存状态，然后设置定时器，在window.getEnd+1的时间的时候进行数据输出
  *
  **/
object A02_HotGItemAnalysis_kafka {

  def main(args: Array[String]): Unit = {

    //1, 获取运行环境
    val senv: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

    //2,获取参数,并验证，验证之后设置到运行环境中，这样可以在flink后台看到
    /*val params: ParameterTool = ParameterTool.fromArgs(args)
    errOutAndExit(params, BOOTSTRAP_SERVERS)
    errOutAndExit(params, GROUP_ID)
    errOutAndExit(params, RETRIES)
    errOutAndExit(params, INPUT_EVENT_TOPIC)
    errOutAndExit(params, INPUT_CONFIG_TOPIC)
    errOutAndExit(params, OUTPUT_TOPIC)
    senv.getConfig.setGlobalJobParameters(params)*/

    //3, 设置事件时间
    senv.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
    senv.setParallelism(1)

    //4.1, 启用检查点，并设置相关的设置
    senv.enableCheckpointing(60000L)
    senv.getCheckpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)
    senv.getCheckpointConfig.setMinPauseBetweenCheckpoints(30000L)
    senv.getCheckpointConfig.setCheckpointTimeout(10000L)
    senv.getCheckpointConfig.enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION)


    //4.2, 设置状态存储方式(此步如果flink配置文件中配置了可不做)
    /*val stateBackend: StateBackend = new RocksDBStateBackend("hdfs://centos01:40010/flink/checkpoints", true)
    senv.setStateBackend(stateBackend)*/


    //4.3, 启用重启策略，这样状态存储才有意义
    senv.setRestartStrategy(RestartStrategies.fixedDelayRestart(10, 30000L))


    //2, source
    val properties = new Properties()
    properties.setProperty("bootstrap.servers", "nebula06:9092")
    properties.setProperty("group.id", "flink_test")
    properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
    properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
    properties.put("auto.offset.reset", "latest")
//    val kafkaDS = senv.addSource(new FlinkKafkaConsumer010[String]("flink01", new SimpleStringSchema(), properties))
//    kafkaDS.print()


    //5, 读取文件
    val value: DataStream[String] = senv
      .addSource(new FlinkKafkaConsumer010[String]("nginx_log_test", new SimpleStringSchema(), properties))


    val stream = value.map(line => line).print()

//    stream.print()

    senv.execute("hotItem")

  }
}

 }


```

