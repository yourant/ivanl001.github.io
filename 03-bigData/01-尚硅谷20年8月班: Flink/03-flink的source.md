[toc]



flink消费kakfa如何保证exactly-once?

flink会维护消费kafka的偏移量，checkpoint机制

代码保证端到端的一致性







## 1, 自定义数据源

* 继承RichParallelSourceFunction， 实现相关方法
* 选择Rich函数是因为有open, close等方法
* 非富函数则没有这些方法

```scala
package org.bool.flink.basic.a02_source

import java.util.Calendar

import org.apache.flink.api.scala.createTypeInformation
import org.apache.flink.streaming.api.functions.source.{RichParallelSourceFunction, SourceFunction}
import org.apache.flink.streaming.api.scala.{DataStream, StreamExecutionEnvironment}

import scala.collection.immutable
import scala.util.Random

/**
 * #author : 不二
 * #date   : 2021/7/25-上午10:45
 * #desc   : 自定义flink source
 * */

object flink_basic_04_custom_source {

  def main(args: Array[String]): Unit = {

    val senv: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    senv.setParallelism(1)

    val sensorDataStream: DataStream[SensorDataModel] = senv.addSource(new SensorSource)
    sensorDataStream.print()

    senv.execute("custom_sensor_source")
  }

  /**
   * 自定义数据源的模型
   * @param id
   * @param timestamp
   * @param temperature
   */
  case class SensorDataModel(id: String,
                             timestamp: Long,
                             temperature: Double)

  /**
   * #author : 不二
   * #date   : 2021/7/25-上午10:44
   * #desc   : 自定义flink的source
   * */
  class SensorSource extends RichParallelSourceFunction[SensorDataModel] {
    var running = true

    override def run(ctx: SourceFunction.SourceContext[SensorDataModel]): Unit = {
      // 自定义source其实很简单
      // 1，从一个渠道获获取数据(比如说mysql等)或者模拟数据
      // 2，把获取到或者模拟的数据通过ctx发送即可。

      val rand = new Random()

      /*var curFTemp: immutable.Seq[(String, Double)] = (1 to 10).map((i: Int) => {
        // 通过高斯噪音产生随即温度值
        ("sensor_" + i, rand.nextGaussian() * 20)
      })*/
      var curFTemp: immutable.Seq[(String, Double)] = (1 to 10).map((i: Int) => {
        // 通过高斯噪音产生随即温度值
        ("sensor_" + i, rand.nextGaussian() * 20)
      })

      // 这里用运行标识
      while (running) {
        curFTemp = curFTemp.map((item: (String, Double)) => {
          (item._1, item._2 + rand.nextGaussian()*0.5)
        })
        val currentTime: Long = Calendar.getInstance().getTimeInMillis

        // 使用上下文发送数据
        curFTemp.foreach((item: (String, Double)) => {
          ctx.collect(SensorDataModel(item._1, currentTime, item._2))
          Thread.sleep(200)
        })
      }
    }
    override def cancel(): Unit = {
      running = false
    }
  }
}
```

