[toc]

## 1, kafka可视化工具

* kafkatool， 支持所有平台

* http://www.kafkatool.com/download.html



## 2, zookeeper可视化工具

* ZooInspector

* https://issues.apache.org/jira/secure/attachment/12436620/ZooInspector.zip





## 3, zk在服务器上连接

```shell
# 开启
/opt/cloudera/parcels/CDH/lib/zookeeper/bin/zkServer.sh start

# 查看zk状态
/opt/cloudera/parcels/CDH/lib/zookeeper/bin/zkServer.sh status


/opt/cloudera/parcels/CDH/lib/zookeeper/bin/zkCli.sh -server nebula03:2181


```

