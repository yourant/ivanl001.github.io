

```sql
set mapreduce.map.memory.mb=5012;
set mapreduce.reduce.memory.mb=5012;
set mapreduce.map.java.opts=-Xmx4g;
set mapreduce.reduce.java.opts=-Xmx4g;








set mapred.max.split.size=10000000;        -- 决定每个map处理的最大的文档大小，单位为B
set mapred.min.split.size.per.node=1;         -- 节点中可以处理的最小的文档大小
set mapred.min.split.size.per.rack=1;         -- 机架中可以处理的最小的文档大小


SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.dynamic.partitions.pernode=100000;
SET hive.exec.max.created.files=655350;
SET mapreduce.job.reduces=20;
SET hive.merge.mapredfiles=TRUE;


set mapreduce.map.memory.mb=2048;
set mapreduce.reduce.memory.mb=2048;
set mapreduce.map.java.opts=-Xmx4g;
set mapreduce.reduce.java.opts=-Xmx4g;


set mapreduce.map.memory.mb=10240;
set mapreduce.reduce.memory.mb=10240;
set mapreduce.map.java.opts=-Xmx10g;
set mapreduce.reduce.java.opts=-Xmx4g;

# 这个是hadoop1的设置
# set mapred.child.java.opts=-Xmx4096m;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.dynamic.partitions.pernode=100000;



SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.dynamic.partitions.pernode=100000;
SET hive.exec.max.created.files=655350;
SET mapreduce.job.reduces=20;
SET hive.merge.mapredfiles=TRUE;


set mapreduce.map.memory.mb=5120;
set mapreduce.reduce.memory.mb=5120;
set mapreduce.map.java.opts=-Xmx4g;
set mapreduce.reduce.java.opts=-Xmx4g;

SET hive.exec.dynamic.partition.mode=nonstrict;
set mapreduce.map.java.opts=-Xmx10g;
set mapreduce.reduce.java.opts=-Xmx10g;
SET hive.exec.max.dynamic.partitions=100000;
SET hive.exec.max.dynamic.partitions.pernode=100000;
insert overwrite table az.dws_paid_order_and_goods partition(report_date)
select * from az.dws_paid_order_and_goods_bak 
where report_date >= '2018-01-01'
and report_date < '2019-01-01';
```



> 参考：
>
> https://community.cloudera.com/t5/Support-Questions/Map-and-Reduce-Error-Java-heap-space/td-p/45874

```java
If you need more details, pls refer below

 

mapred.map.child.java.opts is for Hadoop 1.x

 

Those who are using Hadoop 2.x, pls use the below parameters instead

 

mapreduce.map.java.opts=-Xmx4g         # Note: 4 GB

mapreduce.reduce.java.opts=-Xmx4g     # Note: 4 GB

 

Also when you set java.opts, you need to note two important points

1. It has dependency on memory.mb, so always try to set java.opts upto 80% of memory.mb

2. Follow the "-Xmx4g" format for opt but numerical value for memory.mb

 

mapreduce.map.memory.mb = 5012        #  Note: 5 GB

mapreduce.reduce.memory.mb = 5012    # Note: 5 GB

 

Finally, some organization will not allow you to alter mapred-site.xml directly or via CM. Also we need thease kind of setup only to handle very big tables, so it is not recommanded to alter the configuration only for few tables..so you can do this setup temporarly by following below steps: 

 

1. From HDFS:

HDFS> export HIVE_OPTS="-hiveconf mapreduce.map.memory.mb=5120 -hiveconf mapreduce.reduce.memory.mb=5120 -hiveconf mapreduce.map.java.opts=-Xmx4g -hiveconf mapreduce.reduce.java.opts=-Xmx4g"

2. From Hive:

hive> set mapreduce.map.memory.mb=5120;

hive> set mapreduce.reduce.memory.mb=5120;

hive> set mapreduce.map.java.opts=-Xmx4g;

hive> set mapreduce.reduce.java.opts=-Xmx4g;

 

Note: HIVE_OPTS is to handle only HIVE, if you need similar setup for HADOOP then use HADOOP_OPTS

 

Thanks

Kumar
```

