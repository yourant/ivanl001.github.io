[toc]

```shell
gobblin-mapreduce.sh --conf ../gobblin-config-dir/gobblin-mapreduce.pull --workdir /home/flink/gobblin/gobblin-work-dir
```



## 0, 参考文档

https://gobblin.readthedocs.io/en/latest/Getting-Started/#introduction

https://github.com/apache/incubator-gobblin

https://www.cnblogs.com/cssdongl/p/6094642.html



## 1,  下载源码

* https://github.com/apache/incubator-gobblin/releases

```shell
incubator-gobblin-release-0.14.0.tar.gz
```



## 2, 解压缩并构建cdh5.12.1的hadoop版本gobblin

```shell
tar -zxvf incubator-gobblin-release-0.14.0.tar.gz
cd incubator-gobblin-release-0.14.0

# -x test是跳过测试，因为在构建cdh版本的时候测试总是过不去，所以只能跳过，具体见上方参考内容
./gradlew clean build -PhadoopVersion=2.6.0-cdh5.12.1 -x test

# 可能需要数分钟，构建成功后会在该代码的根目录下产生一个压缩包
# 该包即是压缩包
apache-gobblin-incubating-bin-0.14.0.tar.gz
tar -zxvf apache-gobblin-incubating-bin-0.14.0.tar.gz -C /usr/local

# 解压缩后会在/usr/local目录下多一个gobblin-dist，即gobblin安装内容
ln -s /usr/local/gobblin-dist /usr/local/gobblin

# 配置环境变量
vim /etc/profile
# 注意：如果有必要，可以cat /etc/profile > /root/.bashrc, cat /etc/profile > ~/.bashrc, 


export GOBBLIN_HOME=/usr/local/gobblin
export PATH=$PATH:$GOBBLIN_HOME/bin
export GOBBLIN_JOB_CONFIG_DIR=/usr/local/gobblin/my-conf
export GOBBLIN_WORK_DIR=/usr/local/gobblin/my-work
```



## 3, gobblin的样例demo

* https://gobblin.readthedocs.io/en/latest/Getting-Started/#introduction



## 4, gobblin的kafka-hdfs

> 这个地方mr暂时还没有跑成功，说没有找到gobblin-kafka-source啥的，后续需要看一下

* https://gobblin.readthedocs.io/en/latest/case-studies/Kafka-HDFS-Ingestion/
* https://www.cnblogs.com/cssdongl/p/6121382.html