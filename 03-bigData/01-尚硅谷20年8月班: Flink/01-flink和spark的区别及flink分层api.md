[toc]

## 1, flink简介

flink:  高吞吐，低延迟的 事件驱动 的实时处理框架



## 2, flink和spark主要区别：

* 1, 底层架构不同

  * flink是事件驱动，实时计算，来一条数据计算一条
  * spark是微批处理，延迟相对较高

  

* 2, 时间语义不同

  * flink支持事件时间
  * spark只能用处理时间



## 3, flink相对spark的其他优势: 

* 3, 检查点
  * flink的检查点比spark更加灵活，性能更高
  * spark的话是在每个stage结束后，才能保存检查点



* 4, 保证消费语义
  * flink是把偏移量保存在flink内部，更容易实现端到端的一致性？
  * spark的话需要外部存储，比如说redis等



***重点***

spark是批计算， 将DAG划分为不同的stage，一个stage完成之后才能计算下一个stage

flink是标准的流执行模式，一个事件在一个节点处理完之后可以直接发往下一个节点进行处理





## 4, flink的分层api

flink主要分为如下几层:

* 4, flink sql api

* 3, flink table api

* 2, DataStream api(Stream, Window)

* 1,ProcessFunction api(event, state, time)

越靠近下层是低级API，越具体， 表达能力越丰富， 使用越灵活

越靠近上层是高级API，越抽象。表达含义越简明，使用起来越方便，灵活性有限











