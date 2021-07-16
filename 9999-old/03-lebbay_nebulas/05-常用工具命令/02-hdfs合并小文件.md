[toc]

## 1, 本地上传hdfs时候直接合并小文件

```shell
hadoop fs -appendToFile 1.txt 2.txt hdfs://cdh5/tmp/lxw1234.txt
```



## 2, 从hdfs下载多个小文件到本地的一个文件里面

```shell
hadoop fs -getmerge /flume/event-test/2020-05-05/* local_largefile.txtge
```



## 3, 合并hdfs上的多个小文件

```shell
hadoop fs -cat /flume/event-test/2020-05-05/* | hadoop fs -appendToFile - /flume/event-test/2020-05-05/hdfs_largefile.txt
```

