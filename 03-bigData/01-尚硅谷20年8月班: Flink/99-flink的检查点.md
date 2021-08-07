[toc]



flink的检查点存在哪里

1, 状态后端?



2, 内存



3, 文件系统



4, RocksDB



flink的检查点机制和spark的检查点有啥不同呢

spark-streaming的检查点仅仅是对driver的故障恢复做了数据和原数据的checkpoint。

flink的检查点机制要复杂的多，它采用的是轻量级的分布式快照，实现每个操作符的快照以及循环流在循环中的操作的快照。









