[toc]

> 参考：https://spark.apache.org/docs/2.3.1/streaming-kafka-0-10-integration.html



```scala
package org.bool.mall.batch

import org.apache.commons.codec.StringDecoder
import org.apache.kafka.clients.consumer.{ConsumerConfig, ConsumerRecord}
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.{SparkConf, TaskContext}
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.streaming.kafka010.{CanCommitOffsets, HasOffsetRanges, KafkaUtils, OffsetRange}
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
 * #author : 不二
 * #date   : 2021/6/27-下午3:14
 * #desc   :
 * kafka偏移量提交有如下几种方式：
 * 1, enable.auto.commit=true，这种方式消费者会定时的更新偏移量，不需要任何人工管理。问题：没有任何语意，
 *    这种更新方式不会在乎这个数据是否处理完毕。所以一般不会使用这种
 * 2, 使用kafka的api在数据处理完成之后更新偏移量.因为kafka保存不是事务性的，所以必须要代码保证幂等性
 * 3, 把偏移量保存到mysql或者redis等其他存储中。然后kafka消费的时候，读取偏移量，并从当前偏移量进行消费
 *    保存的时候有两种：3.1, 事务性保存
 *                  3.2, 非事务性保存, 这种的话就需要手动的保证幂等性
 *
 *  参考：https://spark.apache.org/docs/2.3.1/streaming-kafka-0-10-integration.html
 * */

object sparkStreaming_01_kafakReceive {
  def main(args: Array[String]): Unit = {

    val conf: SparkConf = new SparkConf()
      .setMaster("local[*]")
      .setAppName("HighKafka")

    val ssc = new StreamingContext(conf, Seconds(3))

    // enable.auto.commit：这里控制是否使用kafka自己管理偏移量,这里是消费者定时更新的。这种更新方式不会在乎这个数据是否处理完毕。所以一般不会使用这种
    // 1, enable.auto.commit=true，这种方式消费者会定时的更新偏移量，不需要任何人工管理。问题：没有任何语意，
    //    这种更新方式不会在乎这个数据是否处理完毕。所以一般不会使用这种
    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> "centos01:9092,centos02:9092,centos03:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "spark-dev",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )

    val topics = Array("ads")
    val stream: InputDStream[ConsumerRecord[String, String]] = KafkaUtils.createDirectStream[String, String](
      ssc,
      PreferConsistent,
      Subscribe[String, String](topics, kafkaParams)
    )

    val resRDD: DStream[(String, String)] = stream.map(record => (record.key, record.value))

    // 打印数据
    resRDD.map(item => item._2).print()

    // 2, 使用kafka的api在数据处理完成之后更新偏移量.因为kafka保存不是事务性的，所以必须要代码保证幂等性
    stream.foreachRDD { rdd =>
      val offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      // some time later, after outputs have completed
      stream.asInstanceOf[CanCommitOffsets].commitAsync(offsetRanges)
    }


    // * 3, 把偏移量保存到mysql或者redis等其他存储中。然后kafka消费的时候，读取偏移量，并从当前偏移量进行消费
    // *    保存的时候有两种：3.1, 事务性保存
    // *                  3.2, 非事务性保存, 这种的话就需要手动的保证幂等性
    // 完整的代码可以见最后面的注视代码
    stream.foreachRDD { rdd =>
      val offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      rdd.foreachPartition { iter =>
        val o: OffsetRange = offsetRanges(TaskContext.get.partitionId)
        println(s"${o.topic} ${o.partition} ${o.fromOffset} ${o.untilOffset}")
        // 这里保存偏移量，然后kafka读取的时候，先取出偏移量，然后再从偏移量开始消费
      }
    }

    ssc.start()
    ssc.awaitTermination()
  }
}



/* 这里是自己存储偏移量的伪代码
// The details depend on your data store, but the general idea looks like this

// begin from the the offsets committed to the database
val fromOffsets = selectOffsetsFromYourDatabase.map { resultSet =>
  new TopicPartition(resultSet.string("topic"), resultSet.int("partition")) -> resultSet.long("offset")
}.toMap

val stream = KafkaUtils.createDirectStream[String, String](
  streamingContext,
  PreferConsistent,
  Assign[String, String](fromOffsets.keys.toList, kafkaParams, fromOffsets)
)

stream.foreachRDD { rdd =>
  val offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges

  val results = yourCalculation(rdd)

  // begin your transaction

  // update results
  // update offsets where the end of existing offsets matches the beginning of this batch of offsets
  // assert that offsets were updated correctly

  // end your transaction
}
 */

```

