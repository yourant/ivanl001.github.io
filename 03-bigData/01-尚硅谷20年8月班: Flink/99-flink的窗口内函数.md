[toc]





## flink的窗口内函数有两种：

### 1, 增量窗口聚合函数

```scala
package org.bool.flink.basic.a03_datastream_transform

import org.apache.flink.api.common.functions.AggregateFunction
import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.bool.flink.basic.a02_source.{SensorDataModel, SensorSource}
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows
import org.apache.flink.streaming.api.windowing.time.Time

/**
 * #author : 不二
 * #date   : 2021/7/26-下午8:19
 * #desc   : 窗口计算的两种方式：
 * 1，增量聚合(窗口内)
 * 2，全量聚合(窗口内)
 * 我们这里讲一下增量聚合的方式
 *
 * */
object flink_basic_09_window_aggregate {

  def main(args: Array[String]): Unit = {

    //1, get the execution environment
    val senv: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    senv.setParallelism(1)

    //2, get input data by connecting to the socket
    val socketInputDataStream: DataStream[SensorDataModel] = senv
      .addSource(new SensorSource)

    socketInputDataStream
      .keyBy((item: SensorDataModel) => item.id)
      .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
      // 窗口聚合：其实是对相同的key进行聚合
      .aggregate(new AvgTempAggregate())
      .print()

    senv.execute("flink_basic_09_window_aggregate")
  }

  class AvgTempAggregate extends AggregateFunction[SensorDataModel, 
                                                   (String, Long, Double), 
                                                   (String, Double)] {

    // 这里是初始化累加器
    override def createAccumulator(): (String, Long, Double) = {
      ("", 0, 0)
    }

    override def add(in: SensorDataModel, acc: (String, Long, Double)): (String, Long, Double) = {
      // acc代表是累加器，元祖中的三个值分别为：哪个传感器， 所有温度的个数， 所有温度的加和
      // 来了一条温度， 把这个数据记录到累加器中，从而能计算出温度的平均值
      // 来的数据肯定key是相同的，所以只需要把个数加1， 温度加和即可
      //  注意：这种写法有问题：因为初始化的acc的_1是空的， 所以这里要用value的值
      (in.id, acc._2 + 1, acc._3 + in.temperature)
    }

    override def getResult(acc: (String, Long, Double)): (String, Double) = {
      // 这里是获取结果
      // 用总的温度值除以温度的个数， 就是温度的平均值
       (acc._1, acc._3/acc._2)
    }

    override def merge(acc: (String, Long, Double), 
                       acc1: (String, Long, Double)): (String, Long, Double) = {
      // 这个是不同节点上累加器的合并，合并的规则也很简单：个数加和， 温度加和即可
      (acc._1, acc._2 + acc1._2, acc._3 + acc1._3)
    }
  }
}
```



### 2, 全量窗口聚合函数

```scala
package org.bool.flink.basic.a03_datastream_transform

import java.text.SimpleDateFormat
import java.util.Date

import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows
import org.apache.flink.streaming.api.windowing.time.Time
import org.bool.flink.basic.a02_source.{SensorDataModel, SensorSource}
import org.bool.flink.basic.a03_datastream_transform.flink_basic_09_window_aggregate.AvgTempAggregate
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.scala.function.ProcessWindowFunction
import org.apache.flink.streaming.api.windowing.windows.TimeWindow
import org.apache.flink.util.Collector

/**
 * #author : 不二
 * #date   : 2021/7/26-下午8:45
 * #desc   : 窗口计算的两种方式：
 * * 1，增量聚合
 * * 2，全量聚合
 * * 我们这里讲一下全量聚合的方式
 * */

object flink_basic_09_window_fullAggregate {
  def main(args: Array[String]): Unit = {

    //1, get the execution environment
    val senv: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    senv.setParallelism(1)

    //2, get input data by connecting to the socket
    val socketInputDataStream: DataStream[SensorDataModel] = senv
      .addSource(new SensorSource)


    socketInputDataStream
      .keyBy((item: SensorDataModel) => item.id)
      .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
      // 窗口聚合：其实是对相同的key进行聚合， 增量聚合， 各个分区会先聚合完之后，再合并聚合。
      // 也就是有预聚合的
      // .aggregate(new AvgTempAggregate())
      // .print()
      // 这里的elements会记录当前窗口的所有的数据
      // 一个窗口的数据全部记录下来，然后循环遍历进行统计结果
      .process(new fullWindowAggregate())
      .print()

    senv.execute("flink_basic_09_window_aggregate")
  }

  case class AvgInfo(id: String, avgTemp: Double, windowStart: String, windowEnd: String)

  class fullWindowAggregate extends ProcessWindowFunction[SensorDataModel, AvgInfo, String, TimeWindow] {

    val sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")

    // 这里的elements会记录当前窗口的所有的数据
    // 一个窗口的数据全部记录下来，然后循环遍历进行统计结果
    // 这个是在窗口闭合的时候进行调用
    override def process(key: String,
                         context: Context,
                         elements: Iterable[SensorDataModel],
                         out: Collector[AvgInfo]): Unit = {

      val size: Int = elements.size
      var sum: Double  = 0
      for(e <- elements) {
        sum += e.temperature
      }
      val windowStart: Long = context.window.getStart
      val windowEnd: Long = context.window.getEnd
      val startTime = sdf.format(new Date(windowStart))
      val endTime = sdf.format(new Date(windowEnd))

      out.collect(AvgInfo(key, sum/size, startTime, endTime))
    }
  }
}

```



### 3, 前面两种数据可以结合使用

```scala
package im.ivanl001

import java.sql.Timestamp

import org.apache.flink.api.common.functions.AggregateFunction
import org.apache.flink.api.common.restartstrategy.RestartStrategies
import org.apache.flink.api.common.state.{ListState, ListStateDescriptor}
import org.apache.flink.api.java.tuple.{Tuple, Tuple1}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.datastream.DataStreamSink
import org.apache.flink.streaming.api.environment.CheckpointConfig
import org.apache.flink.streaming.api.functions.KeyedProcessFunction
import org.apache.flink.streaming.api.scala.function.WindowFunction
import org.apache.flink.streaming.api.scala.{KeyedStream, StreamExecutionEnvironment, _}
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.streaming.api.windowing.windows.TimeWindow
import org.apache.flink.streaming.api.{CheckpointingMode, TimeCharacteristic}
import org.apache.flink.util.Collector

import scala.collection.mutable.ListBuffer


/**
  * #author      : ivanl001
  * #creator     : 2019-08-23 21:23
  * #description : 时间窗口左笔右开
  *
  *
  **/

case class UserBehavior(userId: Long, itemId:Long, categoryId:Int, behavior:String, timestamp:Long)

case class ItemViewCount(itemId:Long, windowEnd: Long, count: Long)

object A01_HotItemAnalysis_file_toSimulate {

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


    //5, 读取文件
    val stream: DataStreamSink[String] = senv
      .readTextFile("/Users/ivanl001/Documents/00-zhangbuer/learning_code/04-analysis/01-BigData_v2/0103-Flink-z-project/010301-project-sgg-project01/src/main/resources/UserBehavior.csv")
      .map(line => {
        val lineSplited = line.split(",")
        UserBehavior(lineSplited(0).toLong, lineSplited(1).toLong, lineSplited(2).toInt, lineSplited(3), lineSplited(4).toLong)
      })
      //有序的时间的话，这里不需要进行watermark处理，如果担心网络延迟，才需要进行水位线处理
      .assignAscendingTimestamps(_.timestamp * 1000)
      .filter(_.behavior == "pv")
      //这两个居然上面这个不对，打开就爆红
      //.keyBy(_.itemId)
      .keyBy("itemId")
      .timeWindow(Time.hours(1), Time.minutes(5))
      //这个比window的好处是可以预聚合，要不然直接用reduce等应该也行
      //这个是有预聚合的，而不是等一个窗口所有的数据完整了才做处理
      .aggregate(new IMCountAggator(), new IMWindowFunction())
      .keyBy("windowEnd")
      .process(new TopNHotItems(5))
      .print()

    senv.execute("hotItem")

  }

  //自定义聚合函数
  class IMCountAggator extends AggregateFunction[UserBehavior, Long, Long]{

    override def add(value: UserBehavior, accumulator: Long): Long = {
        accumulator+1
    }

    override def createAccumulator(): Long = 0L

    override def getResult(accumulator: Long): Long = accumulator

    override def merge(a: Long, b: Long): Long = {
      a+b
    }
  }

  //自定义实现window函数
  class IMWindowFunction extends WindowFunction[Long, ItemViewCount, Tuple, TimeWindow]{

    override def apply(key: Tuple, window: TimeWindow, input: Iterable[Long], out: Collector[ItemViewCount]): Unit = {
      val itemId = key.asInstanceOf[Tuple1[Long]].f0
      val count = input.iterator.next()

      out.collect(ItemViewCount(itemId, window.getEnd, count))
    }
  }

}

 class TopNHotItems(topSize: Int) extends KeyedProcessFunction[Tuple, ItemViewCount, String]{

   //1, 首先要创建状态
   private var itemState : ListState[ItemViewCount] = _

   override def open(parameters: Configuration): Unit = {
     super.open(parameters)

     //2, 开始的时候先注册描述器，方便后续进行查找
     val itemStateDesc = new ListStateDescriptor[ItemViewCount]("itemState", classOf[ItemViewCount])

     //3, 获取状态
     itemState = getRuntimeContext.getListState(itemStateDesc)


   }

   override def processElement(value: ItemViewCount, ctx: KeyedProcessFunction[Tuple, ItemViewCount, String]#Context, out: Collector[String]): Unit = {

     //4, 加入状态
     itemState.add(value)

     //5, 注册定时器，方便后续进行处理
     ctx.timerService().registerEventTimeTimer(value.windowEnd + 1)

   }

   override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Tuple, ItemViewCount, String]#OnTimerContext, out: Collector[String]): Unit = {

     super.onTimer(timestamp, ctx, out)

     val allItems : ListBuffer[ItemViewCount] = ListBuffer()

     //下面需要隐式类型转换
     import scala.collection.JavaConversions._
     for (item <- itemState.get()){
       allItems += item
     }

     val sortedItems = allItems.sortBy(_.count)(Ordering.Long.reverse).take(topSize)

     // 将排名数据格式化，便于打印输出
     val result: StringBuilder = new StringBuilder
     result.append("====================================\n")
     result.append("时间：").append(new Timestamp(timestamp - 1)).append("\n")

     for( i <- sortedItems.indices ){
       val currentItem: ItemViewCount = sortedItems(i)
       // 输出打印的格式 e.g.  No1：  商品ID=12224  浏览量=2413
       result.append("No").append(i+1).append(":")
         .append("  商品ID=").append(currentItem.itemId)
         .append("  浏览量=").append(currentItem.count).append("\n")
     }
     result.append("====================================\n\n")


     //状态已经拿到，清理空间
     itemState.clear()
     Thread.sleep(1000)

     out.collect(result.toString())
   }
 }
```

