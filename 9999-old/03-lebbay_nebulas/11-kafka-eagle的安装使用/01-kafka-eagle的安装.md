[toc]



## 0, 参考文档

https://www.kafka-eagle.org/

https://docs.kafka-eagle.org/2.env-and-install/2.installing

https://github.com/smartloli/kafka-eagle



## 1, 下载kafka-eagle

https://www.kafka-eagle.org/

```shell
kafka-eagle-bin-1.4.6.tar.gz 
```





## 2, 上传解压缩并安装配置等

```shell
tar -zxvf kafka-eagle-bin-1.4.6.tar.gz -C /usr/local

#  解压缩之后里面居然还有一层
cd /usr/local/kafka-eagle-bin-1.4.6
ls
tar -zxvf /usr/local/kafka-eagle-bin-1.4.6/kafka-eagle-web-1.4.6-bin.tar.gz -C /usr/local

# 然后解压缩之后是kafka-eagle-web-1.4.6
cd /usr/local/kafka-eagle-web-1.4.6

ln -s /usr/local/kafka-eagle-web-1.4.6 /usr/local/kafka-eagle

export KE_HOME=/usr/local/kafka-eagle
export PATH=$PATH:$KE_HOME/bin

# 修改配置文件
vim /usr/local/kafka-eagle/conf/system-config.properties



######################################
# multi zookeeper & kafka cluster list
######################################
kafka.eagle.zk.cluster.alias=nebulas
nebulas.zk.list=nebula03:2181,nebula04:2181,nebula05:2181
# cluster2.zk.list=xdn10:2181,xdn11:2181,xdn12:2181

######################################
# kafka sqlite jdbc driver address
######################################
kafka.eagle.driver=org.sqlite.JDBC
#  修改这个地址并提前创建目录，否则可能会报错找不到目录
kafka.eagle.url=jdbc:sqlite:/data/kafka-eagle/db/ke.db
kafka.eagle.username=root
kafka.eagle.password=www.kafka-eagle.org
```





## 3, 启动

```shell
cd /usr/local/kafka-eagle/bin
chmod +x /usr/local/kafka-eagle/ke.sh
/usr/local/kafka-eagle/ke.sh start
```



## 4, 页面验证

* http://nebula02:8048/ke/

  

