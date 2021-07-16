[toc]

## kafka消费

```shell
# 节点是nebula03， nebula04， nebula05， nebula06， nebula07
kafka-console-producer --broker-list nebula06:9092 --topic az_bounce_data.test
kafka-console-consumer --bootstrap-server nebula03:9092 --topic az_bounce_data.test --from-beginning
kafka-console-consumer --bootstrap-server nebula03:9092 --topic order
kafka-console-consumer --bootstrap-server centos01:9092 --topic order
kafka-console-consumer --bootstrap-server centos01:9092 --topic az_db
kafka-console-consumer --bootstrap-server centos01:9092 --topic azazie_db


kafka-console-consumer --bootstrap-server nebula03:9092 --topic test

kafka-console-consumer --bootstrap-server centos01:9092 --topic kafka_sample

kafka-console-consumer --bootstrap-server nebula03:9092 --topic az_db  --from-beginning
kafka-console-consumer --bootstrap-server nebula03:9092 --topic app_push.test --from-beginning


kafka-console-consumer --bootstrap-server nebula03:9092 --topic app_push.test  --from-beginning


kafka-console-consumer --bootstrap-server nebula03:9092 --topic pageview.test --from-beginning
kafka-console-consumer --bootstrap-server nebula03:9092 --topic pageview --from-beginning

kafka-console-consumer --bootstrap-server nebula03:9092 --topic pageview



kafka-console-consumer --bootstrap-server nebula03:9092 --topic event.test --from-beginning


kafka-console-consumer --bootstrap-server nebula07:9092 --topic timing.test --from-beginning



kafka-console-consumer --bootstrap-server nebula03:9092,nebula04:9092,nebula05:9092,nebula06:9092,nebula07:9092 --topic azazie_db --from-beginning
```



## kafka数据位置

```shell
/var/local/kafka/data
```



kafka-console-producer --broker-list nebula06:9092 --topic app_push.test

 

## kafka生产

```shell
kafka-console-producer --broker-list centos01:9092 --topic order
kafka-console-producer --broker-list centos01:9092 --topic azazie_db

kafka-console-producer --broker-list centos01:9092 --topic kafka_sample

# 节点是nebula03， nebula04， nebula05， nebula06， nebula07
kafka-console-producer --broker-list 172.31.12.253:9092 --topic nginx_log_test


kafka-console-producer --broker-list nebula06:9092 --topic nginx_log_test_01

kafka-console-consumer --bootstrap-server nebula06:9092 --topic nginx_log_test_01




kafka-console-producer --broker-list nebula06:9092 --topic test

```



## kafka创建查看主题

```shell
kafka-topics --create --zookeeper nebula03:2181 --replication-factor 1 --partitions 5 --topic appevent.dev
kafka-topics --list --zookeeper nebula03:2181
# 查看topic的分区等
kafka-topics --zookeeper nebula03:2181 --describe --topic test
```



## kafka查看消费速度

```shell
# 查看topic的消费进度, -1表示最大的消费偏移量(不是consumer的消费进度，应该是producer的生产的最大位移)
kafka-run-class kafka.tools.GetOffsetShell --broker-list nebula03:9092 --topic test --time -1
```



## kafka删除主题

```shell
kafka-topics --zookeeper nebula03:2181 --delete --topic test
```



## kafka手动重分区

* 详细参考：00-01-kafka的分区操作等.md

```shell
# kafka-topics.sh命令下的帮助：
# --replica-assignment  slave02:slave03, slave02:slave03

kafka-topics --alter --zookeeper nebula03:2181 --topic test --replica-assignment 02:03,02:03,02:03,02:03,02:03
```

