## 1, 元聚使用的设置

```mysql
-- 允许动态分区
SET hive.exec.dynamic.partition.mode=nonstrict;
-- 在所有执行MR的节点上，最大一共可以创建多少个动态分区, 默认1000
-- 先注释
-- SET hive.exec.max.dynamic.partitions=100000;
-- 在每个执行MR的节点上，最大可以创建多少个动态分区, 默认100, 源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。
-- 先注释
-- SET hive.exec.max.dynamic.partitions.pernode=100000;
-- 整个MR Job中，最大可以创建多少个HDFS文件，默认100000, 一般默认值足够了，除非你的数据量非常大，需要创建的文件数大于100000，可根据实际情况加以调整
-- 先注释
-- SET hive.exec.max.created.files=655350;
SET mapreduce.job.reduces=20;
-- 这个看后面2优化
SET hive.merge.mapredfiles=TRUE;
```



```mysql
SET hive.exec.dynamic.partition.mode=nonstrict;
SET mapreduce.job.reduces=20;
-- SET hive.merge.mapredfiles=TRUE;
```





## 2, 优化

> 后来发现小文件合并不好使，超过10几M 的时候就很容易产生多个文件

```mysql
-- 在map-only job后合并文件，默认true, 所以不需要设置
-- SET hive.merge.mapfiles = true
-- 在map-reduce job后合并文件，默认false(如果没有reduce过程，也不需要设置这个)
SET hive.merge.mapredfiles = true
-- 合并后每个文件的大小，默认256 000 000, 所以也不需要设置
-- SET hive.merge.size.per.task = 256000000
-- 平均文件大小，是决定是否执行合并操作的阈值，默认16000000
SET hive.merge.smallfiles.avgsize=128000000
```



* 所以结合分区，最终如下

```mysql
SET hive.exec.dynamic.partition.mode=nonstrict;
SET mapreduce.job.reduces=20;
-- 在map-only job后合并文件，默认true, 所以不需要设置
-- SET hive.merge.mapfiles = true
-- 在map-reduce job后合并文件，默认false(如果没有reduce过程，也不需要设置这个)
SET hive.merge.mapredfiles = true;
-- 合并后每个文件的大小，默认256 000 000, 所以也不需要设置
-- SET hive.merge.size.per.task = 256000000
-- 平均文件大小，是决定是否执行合并操作的阈值，默认16 000 000
SET hive.merge.smallfiles.avgsize=128000000;
```



```mysql
SET hive.exec.dynamic.partition.mode=nonstrict;
SET mapreduce.job.reduces=20;
SET hive.merge.mapredfiles = true;
SET hive.merge.smallfiles.avgsize=128000000;
```

