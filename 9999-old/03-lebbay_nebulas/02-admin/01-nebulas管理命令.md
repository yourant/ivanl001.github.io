[toc]



### 1, 开启关闭cm命令

* 注意考虑是否要在关闭cm之前先关闭cdh集群和Cloudera Management Service

#### 1.0, 日志目录

```shell
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-server/cloudera-scm-server.log
/opt/cloudera-manager/cm-5.12.1/log/cloudera-scm-agent/cloudera-scm-agent.log
```

#### 1.1, 开启命令

```shell
# 先切换到root用户下
sudo su

# 开启server，开启各个节点上的agent
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server start
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start
# imcall.sh /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent start
```

#### 1.2, 关闭命令

```shell
# 先切换到root用户下
# 先关闭agent，再关闭server
sudo su
imcall.sh /opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent stop
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server stop
```



