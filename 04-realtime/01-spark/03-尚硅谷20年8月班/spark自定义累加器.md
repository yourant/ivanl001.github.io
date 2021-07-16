[toc]

## 复杂累加器的实现

```scala
package org.bool.mall.batch.acc

import org.apache.spark.util.AccumulatorV2
import org.bool.mall.batch.model.CategoryCountInfo

import scala.collection.mutable

/**
 * #author : 不二
 * #date   : 2021/6/23-上午8:38
 * #desc   : 这个累加器不是简单的数字加和，而是模型与模型之间的累加，相对还蛮复杂的，可以参考
 * */

class CategoryCountInfoAcc extends AccumulatorV2[CategoryCountInfo, mutable.Map[String, CategoryCountInfo]] {

  // 初始化返回值
  var resultMap: mutable.Map[String, CategoryCountInfo] = mutable.Map[String, CategoryCountInfo]()

  override def isZero: Boolean = resultMap.isEmpty

  override def copy(): AccumulatorV2[CategoryCountInfo, mutable.Map[String, CategoryCountInfo]] = {
    val acc = new CategoryCountInfoAcc
    acc.resultMap = this.resultMap
    acc
  }

  override def reset(): Unit = resultMap.clear()

  // 这个是把其他的数据累加到我们已有的数据上(在单个exector上的操作，也就是在单个累加器内部如何进行加和)
  override def add(v: CategoryCountInfo): Unit = {
//    println("aaa")
//    println(v)
    // 从数组里面取出数据
    if (this.resultMap.contains(v.categoryId)) {
      val model: CategoryCountInfo = this.resultMap.getOrElse(v.categoryId, CategoryCountInfo(v.categoryId, 0, 0, 0))
      model.orderCount = model.orderCount + v.orderCount
      model.payCount = model.payCount + v.payCount
      model.clickCount = model.clickCount + v.clickCount
      // this.resultMap(v.categoryId) = model;
      this.resultMap.update(v.categoryId, model)
    } else {
      // 这个是map添加元素的方式
//      this.resultMap += (v.categoryId -> v)
      //this.resultMap.+=(v.categoryId -> v)
      this.resultMap(v.categoryId) = v
    }
  }

  // 合并，在多个exector之间如何进行操作，其实也就是不同的exector上的两个累加器如何进行运算
  override def merge(other: AccumulatorV2[CategoryCountInfo, mutable.Map[String, CategoryCountInfo]]): Unit = {

    val otherMap: mutable.Map[String, CategoryCountInfo] = other.value

    // 这里返回的是resultMap
    otherMap.foldLeft(this.resultMap){
      (resultMap, otherKV) => {
        if(resultMap.contains(otherKV._1)){
          val otherModel = otherKV._2
          val info: CategoryCountInfo = resultMap.getOrElse(otherKV._1, CategoryCountInfo(otherKV._1, 0, 0, 0))
          info.clickCount += otherModel.clickCount
          info.payCount += otherModel.payCount
          info.orderCount += otherModel.orderCount
          resultMap.update(otherKV._1, info)
        } else {
          // resultMap(otherKV._1) = otherKV._2
          resultMap += otherKV
        }
        resultMap
      }
    }

    // 这里返回的是otherMap，还需要再赋值回去
    /*val result: mutable.Map[String, CategoryCountInfo] = this.resultMap.foldLeft(otherMap) {
      (other, kv) => {
        val key: String = kv._1
        val valueModel: CategoryCountInfo = kv._2

        // 需要从other里面拿出key对应的模型，然后跟现有模型进行加和，然后再把map返回
        if (other.contains(key)) {
          // 如果other中已经包含，那么需要加和，并更新到other上
          val theModel: CategoryCountInfo = other.getOrElse(key, CategoryCountInfo(key, 0, 0, 0))
          theModel.clickCount += valueModel.clickCount
          theModel.payCount += valueModel.payCount
          theModel.orderCount += valueModel.orderCount
          other.update(key, theModel)
        } else {
          // 如果other上没有，那么直接把对应的数据添加上即可
          other += kv
        }
        other
      }
    }
    // 注意：这里要进行赋值回去
    this.resultMap = result*/
  }

  override def value: mutable.Map[String, CategoryCountInfo] = {
    this.resultMap
  }
}
```

