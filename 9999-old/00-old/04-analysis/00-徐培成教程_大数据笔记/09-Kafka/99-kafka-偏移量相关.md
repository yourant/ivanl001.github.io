

参考：https://zhuanlan.zhihu.com/p/353359319

https://blog.csdn.net/lizhitao/article/details/41441513



```shell
# 查看
kafka-run-class kafka.tools.GetOffsetShell --broker-list nebula03:9092,nebula04:9092,nebula05:9092 --topic pageview --time -1


kafka-run-class kafka.tools.GetOffsetShell --broker-list centos01:9092,centos02:9092,centos03:9092 --topic maxwell --time -1


# 查看某个topic的分区和副本以及leader
kafka-topics --zookeeper nebula04:2181 --describe --topic pageview




# kafka修复leader 倾斜问题
kafka-preferred-replica-election  --zookeeper nebula03:2181 --path-to-json-file topicPartitionList.json
# topicPartitionList.json文件如下：
# 给出具体的topic和分区即可，执行命令后会重新选举leader
{
 "partitions":
  [
    {"topic":"pageview","partition": 3}
  ]
}


# kafka增加副本数量
kafka-reassign-partitions --zookeeper nebula03:2181 --reassignment-json-file event_reassignment.json --execute
# reassignment.json文件内容如下：
{
    "version": 1,
    "partitions": [
        {
            "topic": "event",
            "partition": 0,
            "replicas": [
                169,
                170
            ]
        },
        {
            "topic": "event",
            "partition": 1,
            "replicas": [
                170,
                171
            ]
        },
        {
            "topic": "event",
            "partition": 2,
            "replicas": [
                171,
                167
            ]
        },
        {
            "topic": "event",
            "partition": 3,
            "replicas": [
                167,
                168
            ]
        },
        {
            "topic": "event",
            "partition": 4,
            "replicas": [
                168,
                169
            ]
        }
    ]
}

# ------------- # 成功后会有如下提示：-------------
Current partition replica assignment

{"version":1,"partitions":[{"topic":"event","partition":2,"replicas":[171],"log_dirs":["any"]},{"topic":"event","partition":1,"replicas":[170],"log_dirs":["any"]},{"topic":"event","partition":3,"replicas":[167],"log_dirs":["any"]},{"topic":"event","partition":0,"replicas":[169],"log_dirs":["any"]},{"topic":"event","partition":4,"replicas":[168],"log_dirs":["any"]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions.
```



