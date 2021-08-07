[toc]

## 1, flink 提供了8个ProcessFunction：



### 1, ProcessFunction<I, O>

```shell
# 可以处理不开窗不keyby的情况
# 这个就是数据来一条，处理一次
# processElement:每来一个元素处理一次
```

```scala
// ProcessFunction处理的是没有keyby的数据
class FreezingAlarmFunction extends ProcessFunction[SensorDataModel, SensorDataModel] {

  // 侧输出标签，后续可以根据这个标签拿到侧输出的内容
  lazy val freezingAlarmMark = new OutputTag[String]("freezing-alarm")

  override def processElement(value: SensorDataModel,
                              ctx: ProcessFunction[SensorDataModel, SensorDataModel]#Context,
                              out: Collector[SensorDataModel]): Unit = {

    if (value.temperature < 23.0) {
      ctx.output(freezingAlarmMark, s"${value.id}的传感器低温报警, 温度为:${value.temperature}")
    }
    out.collect(value)
  }
}
```

### 2, ProcessWindowFunction[IN, OUT, KEY, W <: Window]

```shell
# 这个要用在有keyby的开窗上,一次统计某个key的所有数据
# 里面没有processElement方法，只有process，只会在窗口闭合时候调用一次
# process：只会在窗口闭合的时候调用一次
```

```scala
class Count extends ProcessWindowFunction[(String, Long), String, String, TimeWindow] {
  override def process(key: String,
                       context: Context, 
                       elements: Iterable[(String, Long)],
                       out: Collector[String]): Unit = {
    out.collect(s"窗口中共有: ${elements.size} 个元素")
  }
}
```



### 3, ProcessAllWindowFunction[IN, OUT, W <: Window]

```shell
# 这个要用在没有keyby的开窗上，一次统计窗口里的所有数据
# 里面没有processElement方法，只有process，只会在窗口闭合时候调用一次
# process：只会在窗口闭合的时候调用一次
```

```scala
class WindowResult extends ProcessAllWindowFunction[Long, String, TimeWindow] {
  override def process(context: Context,
                       elements: Iterable[Long],
                       out: Collector[String]): Unit = {
    // 因为有过预聚合，所以只会有一个值
    out.collect("窗口结束时间：" + new Timestamp(context.window.getEnd) + "的pv统计值是：" + elements.head)
  }
}
```



### 4, KeyedProcessFunction<K, I, O>

```shell
# 仅仅keyby, 没有开窗 的情况
# -----------------------在相同的processElement里面都是相同key的元素-----------------------
# processElement:每来一个元素处理一次
```

```scala
class IMKeyedFunction extends KeyedProcessFunction[String, (String, Long), String] {
  val sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
  // 每来一条数据处理一次
  // 这个和ProcessWindowFunction里面的process不太一样，那个只会调用一次
  override def processElement(value: (String, Long),
                              ctx: KeyedProcessFunction[String, (String, Long), String]#Context,
                              out: Collector[String]): Unit = {

    // 注册定时器, 当前时间10s之后执行
    // 这里要加上水位线延迟的时间
    ctx.timerService().registerEventTimeTimer(value._2 + 10*1000L)

    // out.collect("来了一条数据")
    val currentProcessingTime: String = sdf.format(new Date(ctx.timerService().currentProcessingTime()))
    out.collect("来了一条数据，currentWatermark： "
                + ctx.timerService().currentWatermark()
                + ",currentProcessingTime:" + currentProcessingTime
                + "触发时间为：" + (value._2 + 10*1000L))
  }

  override def onTimer(timestamp: Long,
                       ctx: KeyedProcessFunction[String, (String, Long), String]#OnTimerContext,
                       out: Collector[String]): Unit = {

    val currentProcessingTime: String = sdf.format(new Date(ctx.timerService().currentProcessingTime()))
    val timeStr: String = sdf.format(new Date(timestamp))
    // super.onTimer(timestamp, ctx, out)
    out.collect(" 定时器触发了-----currentProcessingTime：" + currentProcessingTime + "， timestamp：" + timeStr)
  }
}
```



### 5, CoProcessFunction<IN1, IN2, OUT> 

```shell
# -----------------------在相同的processElement里面都是相同key的元素-----------------------
# 处理connect之后的流可以用这个
# 会有两个处理方法分别处理两个流的数据：processElement1和processElement2
# 比如说一条流可以控制另外一条流的数据。温度开关等等
# processElement1/processElement2：每来一个元素处理一次
```

```scala
class MatchFunction extends CoProcessFunction[OrderInfo, PayInfo, String] {
  
  lazy val orderState: ValueState[OrderInfo] = getRuntimeContext.getState(
    new ValueStateDescriptor[OrderInfo]("order-state", Types.of[OrderInfo]))

  lazy val payState: ValueState[PayInfo] = getRuntimeContext.getState(
    new ValueStateDescriptor[PayInfo]("pay-state", Types.of[PayInfo]))

  override def processElement1(value: OrderInfo,
                               ctx: CoProcessFunction[OrderInfo, PayInfo, String]#Context,
                               out: Collector[String]): Unit = {

    // 进入到函数里面的时候一定是相同的key
    val pay: PayInfo = payState.value()
    // 说明支付的信息已经先到了

    if (value.orderType == "order" && pay != null) {
      // 说明已经对上了，就不需要保存状态，而且已有状态还可以清空
      payState.clear()
      out.collect(s"订单：${value.orderId}的订单对账成功 ")

      // 这里应该把定时器给取消吧，因为更新状态的时候是设置有定时器的
      // 支付来了，那么就可以把定时器清空了
      ctx.timerService().deleteEventTimeTimer(pay.payTime + 5000L)

    } else {
      // 说明支付事件没到，保存订单事件
      orderState.update(value)
      // 并设置定时器进行限定时间更新 数据, 允许5s的时间间隔
      ctx.timerService().registerEventTimeTimer(value.orderTime + 5000L)
    }
  }

  override def processElement2(value: PayInfo,
                               ctx: CoProcessFunction[OrderInfo, PayInfo, String]#Context,
                               out: Collector[String]): Unit = {
    val order: OrderInfo = orderState.value()
    if (value.payType == "pay" && order != null) {
      out.collect(s"订单：${value.orderId}的订单对账成功 ")
      // 数据来了之后，另外一个状态会被清空，当前数据也不会保存到状态里面
      orderState.clear()
      // 支付来了，那么就可以把定时器清空了
      ctx.timerService().deleteEventTimeTimer(order.orderTime + 5000L)
    } else {
      // 说明支付事件没到，保存订单事件
      payState.update(value)
      // 并设置定时器进行限定时间更新 数据, 允许5s的时间间隔
      ctx.timerService().registerEventTimeTimer(value.payTime + 5000L)
    }
  }

  override def onTimer(timestamp: Long,
                       ctx: CoProcessFunction[OrderInfo, PayInfo, String]#OnTimerContext,
                       out: Collector[String]): Unit = {
    // println("进入到定时器")

    if (orderState.value() != null) {
      // 说明订单5s后还是没来
      ctx.output(unMatchedOrders, s"订单${orderState.value().orderId} 对账失败，支付超时")
      orderState.clear()
    } else if (payState.value() != null) {
      // 说明订单5s后还是没来
      ctx.output(unMatchedPays, s"订单${payState.value().orderId} 对账失败，下单超时")
      payState.clear()
    } else {
      // 这里现在肯定不会再来了。因为在规定时间内到的数据，会把状态给清空到
      // 这里其实就没必须要了，如果是这种情况，可以直接取消定时器了
      println("两个都到了-----")
      /*payState.clear()
        orderState.clear()*/
    }
  }
}
```



### 6, ProcessJoinFunction<IN1, IN2, OUT>

```shell
# join之后只有一条流哈，这是join和connect的不同，connect之后有两条流
# 
```

```scala
class IMProcessJoinFunction extends ProcessJoinFunction[(String, String, Long), 
                                                        (String, String, Long), String] {
  override def processElement(left: (String, String, Long),
                              right: (String, String, Long),
                              ctx: ProcessJoinFunction[(String, String, Long), 
                                                       (String, String, Long), String]#Context,
                              out: Collector[String]): Unit = {
    out.collect("left:" + left + " ---> " + "right:" + right)
  }
}
```





### 7, BroadcastProcessFunction

```shell
# 暂时未用过
```





### 8, KeyedBroadcastProcessFunction

```shell
# 暂时未用过
```



## 2, ProcessFunction的底层trigger实现

```shell
# ---------trigger是窗口聚合函数的底层实现--------
Trigger：
窗口聚合函数的底层实现，可以自由的控制窗口的时机
```



```scala
class OneSecondsIntervalTrigger extends Trigger[(String, Long), TimeWindow] {

  override def onElement(element: (String, Long),
                         timestamp: Long,
                         window: TimeWindow,
                         ctx: Trigger.TriggerContext): TriggerResult = {
    // 注意：如果这里自己设置了定时器， 那么默认的会调用过一次， 定时器的还会调用一次
    TriggerResult.CONTINUE
  }

  override def onProcessingTime(time: Long,
                                window: TimeWindow,
                                ctx: Trigger.TriggerContext): TriggerResult = {
    println("onProcessingTime---触发～～～" + time)
    TriggerResult.CONTINUE
  }

  override def onEventTime(time: Long,
                           window: TimeWindow,
                           ctx: Trigger.TriggerContext): TriggerResult = {
    println("onEventTime---触发～～～" + time)
    TriggerResult.CONTINUE
  }

  override def clear(window: TimeWindow,
                     ctx: Trigger.TriggerContext): Unit = {}
}
```



### AggregateFunction是干啥的？

```shell
AggregateFunction
# 增量聚合函数
# 这个函数和processFunction不太一样，只是给出数据， 进行聚合，并不涉及到任何上下文信息
# 这个函数只有输入值和输出值两个参数
```



```shell
# 这个函数和processFunction不太一样，只是给出数据， 进行聚合，并不涉及到任何上下文信息
# 这个函数只有输入值和输出值参数
JoinFunction
```


