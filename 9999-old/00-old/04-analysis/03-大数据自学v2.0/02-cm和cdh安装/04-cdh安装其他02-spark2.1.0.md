> 版本号规则：
>
> **<CDS_version>.cloudera<release_number>-1.<cdh_build_version>.p<patch_version>.<build_number>**
>
> 2.1.0.cloudera3-1.cdh5.13.3.p0.569822



cdh2.12.1安装其他版本的spark，这里是spark2.1.0，官网教程：

https://www.cloudera.com/documentation/spark2/latest/topics/spark2_installing.html



## 1, 下载所需要的文件

* SPARK2_ON_YARN-2.1.0.cloudera1.jar
* SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904-el7.parcel
* SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904-el7.parcel.sha1

下载地址：

http://archive.cloudera.com/spark2/csd/SPARK2_ON_YARN-2.1.0.cloudera1.jar

http://archive.cloudera.com/spark2/parcels/2.1.0.cloudera1/









http://archive.cloudera.com/spark2/csd/SPARK2_ON_YARN-2.2.0.cloudera1.jar

http://archive.cloudera.com/spark2/parcels/2.2.0.cloudera1/SPARK2-2.2.0.cloudera1-1.cdh5.12.0.p0.142354-el7.parcel

http://archive.cloudera.com/spark2/parcels/2.2.0.cloudera1/SPARK2-2.2.0.cloudera1-1.cdh5.12.0.p0.142354-el7.parcel.sha1







## 2, 把jar包文件放到/opt/cloudera/csd/文件夹下

⚠️：此步必须要做，而且在放置所有文件后，需要重启cloudera-scm-server的server，不然的话，后面添加服务的时候是找不到spark2这项服务的

```shell
mv SPARK2_ON_YARN-2.1.0.cloudera1.jar /opt/cloudera/csd
```



## 3, 把.parcel和.sha1放到parcel-repo目录下

⚠️：**.sha1**文件需要改后缀名为.sha, 去掉最后的那个1

```shell
mv SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904-el7.parcel /opt/cloudera/parcel-repo
mv SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904-el7.parcel.sha1 /opt/cloudera/parcel-repo/SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904-el7.parcel.sha
```



## 4, 页面刷新parcel

![image-20190807171335727](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20190807171335727.png)

## 5, 分配激活即可

![image-20190807171440542](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20190807171440542.png)



## 6, 重启cm哦！！！

```shell
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-server restart
/opt/cloudera-manager/cm-5.12.1/etc/init.d/cloudera-scm-agent restart
```



## 7, 然后集群安装

### 7.1, 注意安装完更改默认的yarn的资源配置,如下：

```shell
# 单个任务可申请的最多物理内存量，我设置的是2G
yarn.scheduler.maximum-allocation-mb

# 每个节点上yarn可以使用的物理内存：我设置的是2G，也就是每台nodemanager可以支配的内存有多少
# 因为设置的是2G，那么我一共三台服务器配置nodemanager，所以一共6G内存
yarn.nodemanager.resource.memory-mb

# cpu核数，根据具体而定，我设定5个
# 可以为容器分配的虚拟 CPU 内核的数量。该参数在 CDH 4.4 以前版本中无效。
# 我集群中设置了3个
yarn.nodemanager.resource.cpu-vcores
```



## 8, 启动spark报错

* 注意：这一步必须要解决，不然后续启动spark2-shell的时候，不会使用yarn模式
* 如果不解决，直接启动，也是能启动的，但是spark2-shell后续使用的时候是有问题的，就是启动的时候不会提交到yarn上

```java
Error: JAVA_HOME is not set and could not be found.
```



> 问题原因：最后找到原因是因为CM安装Spark不会去环境变量去找Java，需要将Java路径添加到CM配置文件



### 8.1, 解决方法1（需要重启cdh）：

[root@hadoop102 java]# vim /opt/module/cm/cm-5.12.1/lib64/cmf/service/client/deploy-cc.sh

在文件最后加上

```java
JAVA_HOME=/usr/local/jdk
export JAVA_HOME=/usr/local/jdk
```



### 8.2, 解决方法2（无需重启cdh）：

cdh不会使用系统默认的JAVA_HOME环境变量，而是依照bigtop进行管理

查看/opt/cloudera-manager/cm-5.12.1/lib64/cmf/service/common/cloudera-config.sh中可知：

![image-20190807174007291](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20190807174007291.png)

bigtop中java的环境变量是：/usr/java/jdk1.8



因此我们需要在指定的/usr/java/jdk1.8目录下安装jdk。当然我们已经在/opt/module/jdk1.8.0_144下安装了jdk，因此创建一个连接过去即可

```shell
ln -s /usr/local/jdk /usr/java/jdk1.8
```



### 8.3, 解决方法3(需要重启cdh)：

找到hadoop102、hadoop103、hadoop104三台机器的配置，配置java主目录

![image-20190807174231389](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20190807174231389.png)





SPARK2_ON_YARN-2.1.0.cloudera1.jar

SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904-el7.parcel

SPARK2-2.1.0.cloudera1-1.cdh5.7.0.p0.120904-el7.parcel.sha



## 9, 安装完成！



## 10, 错误解决

### 10.1, NoClassDefFoundError: FSDataInputStream

```java
// 在部分节点上报错：
// 之所以说部分，是因为cdh中安装spark2，实际上页面上就一个history server，我把history server安装在centos01上，也只有centos01上不报错，可以直接登入。所以可能其他节点本身就不推荐吧。后续看看。
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/hadoop/fs/FSDataInputStream
```

参考：https://spark.apache.org/docs/latest/hadoop-provided.html

​			https://www.cnblogs.com/yanghuabin/p/8329205.html



解决办法：

```shell
vim /opt/cloudera/parcels/SPARK2/lib/spark2/bin/load-spark-env.sh

# 添加 
# 这里实际上就是执行一条hadoop的命令，将hadoop的classpath引一下
export SPARK_DIST_CLASSPATH=$(${HADOOP_HOME}/bin/hadoop classpath)
export SPARK_DIST_CLASSPATH=$(hadoop --config /path/to/configs classpath)
/etc/hadoop/conf

### 只添加这里即可
export SPARK_DIST_CLASSPATH=$(hadoop --config /etc/hadoop/conf classpath)
export HADOOP_CONF_DIR=/etc/hadoop/conf
export YARN_CONF_DIR=/etc/hadoop/conf

```



### 10.2, Failed to get database global_temp

```java
// 这个是history server同一个节点上spark2-shell时候警告，不知道什么原因，不影响使用，后续再看
WARN metastore.ObjectStore: Failed to get database global_temp, returning NoSuchObjectException
```



### 10.3, Required executor memory (1024+384 MB) is above the max threshold

https://blog.csdn.net/sunfect/article/details/86031857

```java
启动Spark-Shell报错：
java.lang.IllegalArgumentException: Required executor memory (1024+384 MB) is above the max threshold (1024 MB) of this cluster! Please check the values of ‘yarn.scheduler.maximum-allocation-mb’ and/or ‘yarn.nodemanager.resource.memory-mb’.
```



