[TOC]

> 参考文章：
>
> https://www.jianshu.com/p/6299048fad66
>



## 1, 需要开启mysql基于binlog主从备份

> 具体开启方法参考数据库mysql部分



## 2, 单机版canal安装

### 01，在mysql的master中给canal赋予权限

```mysql
grant all privileges on *.* to 'canal'@'%' identified by 'canal';
flush privileges;

select user, host, password from mysql.user;
```



### 02，下载解压

*　此处略过





### 03, 配置canal.properties

* 单机模式下该配置文件不需要改动



### 04, 配置instance.properties

* 改配置文件在example下

```shell
canal.instance.mysql.slaveId=20

# position info
#  这个地方要改成mysql的master地址
canal.instance.master.address=centos01:3306

# 给canal赋予权限的时候的账号密码是什么，这里就填写什么，默认就是canal
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
```



### 05，启动关闭canal

```shell
# 启动canal
/usr/local/canal/bin/startup.sh

# 关闭canal
/usr/local/canal/bin/stop.sh

# 日志相关位置
less /usr/local/canal/logs/canal/canal.log
less /usr/local/canal/logs/example/example.log
```



## 3, 集群版canal

### 01，在mysql的master中给canal赋予权限

```mysql
grant all privileges on *.* to 'canal'@'%' identified by 'canal';
flush privileges;

select user, host, password from mysql.user;
```



### 02，下载解压

- 此处略过



### 03, 配置canal.properties

```shell
vim /usr/local/canal/conf/canal.properties


# 配置zk，方便选举leader
canal.zkServers = centos01:2181,centos02:2181,centos03:2181

# 开启这一项
canal.instance.global.spring.xml = classpath:spring/default-instance.xml
```



### 04, 配置instance.properties

- 改配置文件在example下

```shell
vim /usr/local/canal/conf/example/instance.properties


canal.instance.mysql.slaveId=20

# position info
#  这个地方要改成mysql的master地址
canal.instance.master.address=centos01:3306

# 给canal赋予权限的时候的账号密码是什么，这里就填写什么，默认就是canal
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
```



### 05，启动关闭canal

* 在主节点和从节点上分别启动即可

```shell
# 启动canal
/usr/local/canal/bin/startup.sh

# 关闭canal
/usr/local/canal/bin/stop.sh

# 日志相关位置
less /usr/local/canal/logs/canal/canal.log
less /usr/local/canal/logs/example/example.log
```



### 06, zk验证

```shell
# zk客户端中获取
get /otter/canal/destinations/example/running

# 结果如下：说明主节点是101，也就是centos01
{"active":true,"address":"192.168.215.101:11111"}
```



## 4，properties配置文件参数解释

> properties配置分为两部分：

```shell
canal.properties  (系统根配置文件)
instance.properties  (instance级别的配置文件，每个instance一份)
```

### 01, canal.properties

```shell
canal.destinations  #当前server上部署的instance列表 
canal.conf.dir  #conf/目录所在的路径   
canal.auto.scan #开启instance自动扫描
#如果配置为true，canal.conf.dir目录下的instance配置变化会自动触发：
#a. instance目录新增： 触发instance配置载入，lazy为true时则自动启动
#b. instance目录删除：卸载对应instance配置，如已启动则进行关闭
#c. instance.properties文件变化：reload instance配置，如已启动自动进行重启操作
canal.auto.scan.interval    #instance自动扫描的间隔时间，单位秒  
canal.instance.global.mode  #全局配置加载方式   
canal.instance.global.lazy  #全局lazy模式   
canal.instance.global.manager.address   #全局的manager配置方式的链接信息    无
canal.instance.global.spring.xml    #全局的spring配置方式的组件文件 
canal.instance.example.mode
canal.instance.example.lazy
canal.instance.example.spring.xml
#instance级别的配置定义，如有配置，会自动覆盖全局配置定义模式
#命名规则：canal.instance.{name}.xxx 无
```



### 02, instance.properties

```shell
canal.id    #每个canal server实例的唯一标识，暂无实际意义   
canal.ip    #canal server绑定的本地IP信息，如果不配置，默认选择一个本机IP进行启动服务   
canal.port  #canal server提供socket服务的端口  
canal.zkServers #canal server链接zookeeper集群的链接信息
#例子：10.20.144.22:2181,10.20.144.51:2181 
canal.zookeeper.flush.period    #canal持久化数据到zookeeper上的更新频率，单位毫秒    
canal.instance.memory.batch.mode    #canal内存store中数据缓存模式
#1. ITEMSIZE : 根据buffer.size进行限制，只限制记录的数量
#2. MEMSIZE : 根据buffer.size  * buffer.memunit的大小，限制缓存记录的大小  
canal.instance.memory.buffer.size   #canal内存store中可缓存buffer记录数，需要为2的指数  
canal.instance.memory.buffer.memunit    #内存记录的单位大小，默认1KB，和buffer.size组合决定最终的内存使用大小  
canal.instance.transactionn.size    最大事务完整解析的长度支持
超过该长度后，一个事务可能会被拆分成多次提交到canal store中，无法保证事务的完整可见性
canal.instance.fallbackIntervalInSeconds    #canal发生mysql切换时，在新的mysql库上查找binlog时需要往前查找的时间，单位秒
#说明：mysql主备库可能存在解析延迟或者时钟不统一，需要回退一段时间，保证数据不丢
canal.instance.detecting.enable #是否开启心跳检查
canal.instance.detecting.sql    #心跳检查sql    insert into retl.xdual values(1,now()) on duplicate key update x=now()
canal.instance.detecting.interval.time  #心跳检查频率，单位秒
canal.instance.detecting.retry.threshold    #心跳检查失败重试次数
canal.instance.detecting.heartbeatHaEnable  #心跳检查失败后，是否开启自动mysql自动切换
#说明：比如心跳检查失败超过阀值后，如果该配置为true，canal就会自动链到mysql备库获取binlog数据
canal.instance.network.receiveBufferSize    #网络链接参数，SocketOptions.SO_RCVBUF 
canal.instance.network.sendBufferSize   #网络链接参数，SocketOptions.SO_SNDBUF 
canal.instance.network.soTimeout    #网络链接参数，SocketOptions.SO_TIMEOUT
```



## 5，错误解决

### 01，pid相关

> found canal.pid , Please run stop.sh first ,then startup.sh

```shell
# 因为遗产关闭，pid文件没有被删除，也就是在/canal/bin目录下会有一个pid相关的文件，可以直接关闭一下，然后重新开启就可以了，也可以直接删掉重启
# 先关闭一下，会自动删除pid文件
/usr/local/canal/bin/stop.sh
# 再重新启动
/usr/local/canal/bin/startup.sh
```



```mysql
source /usr/local/canal-admin/conf/canal_manager.sql
```



http://192.168.215.101:8089/#/login?redirect=%2Fdashboard



```shell
# jps查不出来，应该不是java应用
/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml > /dev/null 2>&1 &
```



```shell
/usr/local/grafana/bin/grafana-server web > /dev/null 2>&1 &
```



## 6, canal直接写入到kafka

```mysql
vim /usr/local/canal/conf/canal.properties


# 配置zk，方便选举leader
canal.zkServers = centos01:2181,centos02:2181,centos03:2181

# ...
# 可选项: tcp(默认), kafka, RocketMQ
canal.serverMode = kafka
# ...
# kafka/rocketmq 集群配置: 192.168.1.117:9092,192.168.1.118:9092,192.168.1.119:9092 
canal.mq.servers = centos01:9092,centos02:9092,centos03:9092
canal.mq.retries = 0
# flagMessage模式下可以调大该值, 但不要超过MQ消息体大小上限
canal.mq.batchSize = 16384
canal.mq.maxRequestSize = 1048576
# flatMessage模式下请将该值改大, 建议50-200
canal.mq.lingerMs = 1
canal.mq.bufferMemory = 33554432
# Canal的batch size, 默认50K, 由于kafka最大消息体限制请勿超过1M(900K以下)
canal.mq.canalBatchSize = 50
# Canal get数据的超时时间, 单位: 毫秒, 空为不限超时
canal.mq.canalGetTimeout = 100
# 是否为flat json格式对象
canal.mq.flatMessage = true
canal.mq.compressionType = none
canal.mq.acks = all
# kafka消息投递是否使用事务
canal.mq.transaction = false
```



```mysql
vim /usr/local/canal/conf/example/instance.properties


canal.instance.mysql.slaveId=20

# position info
# 这个地方要改成mysql的master地址
canal.instance.master.address=centos01:3306

# 给canal赋予权限的时候的账号密码是什么，这里就填写什么，默认就是canal
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal

canal.mq.topic=example
canal.mq.partition=0
```



```mysql
# 启动canal
/usr/local/canal/bin/startup.sh

# 关闭canal
/usr/local/canal/bin/stop.sh

# 日志相关位置
less /usr/local/canal/logs/canal/canal.log
less /usr/local/canal/logs/example/example.log
```

