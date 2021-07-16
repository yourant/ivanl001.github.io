​	

使用imply进行安装

https://imply.io/get-started





参考：

https://blog.csdn.net/qq_21383435/article/details/82707527

https://juejin.im/post/5cc00e4bf265da0368145e45



```shell
#  这个是官网教程，不过过于麻烦
http://druidio.cn/docs/0.9.0/tutorials/cluster.html
```





```shell
nebula03:9092,nebula04:9092,nebula05:9092,nebula06:9092,nebula07:9092
```



```shell
# 查询时候指定时区
{
  "query" : "select count(distinct cid) from pageview where __time >= '2020-05-23'",
  "context" : {
    "sqlTimeZone" : "America/Los_Angeles"
  }
}
```







```shell
lsof -i :8090 | grep LISTEN
ps -ef pid



superset连接druid
https://stackoverflow.com/questions/50182494/add-a-druid-cluster-as-a-sql-database-in-apache-superset

druid://localhost:8082/druid/v2/sql/
```



```shell
/usr/local/druid/bin/supervise  -c /usr/local/druid/conf/supervise/quickstart.conf


nohup /usr/local/druid/bin/supervise  -c /usr/local/druid/conf/supervise/quickstart.conf >> /data/druid/logs/log.txt 2>&1 &


/usr/local/druid/bin/service --down

/usr/local/druid/bin/supervise  -c /usr/local/druid/conf/supervise/quickstart.conf




nebula03:9092,nebula04:9092,nebula05:9092,nebula06:9092,nebula07:9092




/usr/local/druid/bin/service --down
```





http://nebula02:9095/console/druid/#ingestion

http://nebula02:9095/datasets





http://cdh2.opsfun.com:9095/datasets



