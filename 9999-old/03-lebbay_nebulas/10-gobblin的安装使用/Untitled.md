```shell
gobblin-standalone.sh start

gobblin-standalone.sh stop



gobblin-mapreduce.sh --conf /usr/local/gobblin/my-conf/kafka2hdfs_mr.pull


tail -f /usr/local/gobblin/logs/gobblin-current.log




./gradlew clean build -PhadoopVersion=2.6.0-cdh5.12.1 -x test




gobblin-mapreduce.sh --conf ../gobblin-config-dir/gobblin-mapreduce.pull --workdir /home/flink/gobblin/gobblin-work-dir
```



https://gobblin.readthedocs.io/en/latest/Getting-Started/#building-a-distribution



https://gobblin.readthedocs.io/en/latest/user-guide/FAQs/#how-do-i-compile-gobblin-against-cdh



https://www.cnblogs.com/cssdongl/p/6094642.html