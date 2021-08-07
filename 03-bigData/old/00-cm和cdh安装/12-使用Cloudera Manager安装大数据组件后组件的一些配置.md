# 使用Cloudera Manager安装大数据组件后组件的一些配置



> 参考：https://blog.terminus.io/initialize-big-data-stack-with-cloudera-manager/#



### 1. Oozie 相关配置

> 1、上传自定义UDF，后者自定义的节点 (jar)

```
Cloudera Manager平台上传到安装oozie那台机器的 `/var/lib/oozie/` 目录  
```

- 注: `非Cloudera Manager平台`直接上传到 `$OOZIE_HOME/oozie-server/webapps/oozie/WEB-INF/lib/` 即可。

> 2、修改Cloudera Manager平台配置

1. 前往平台的 oozie > 配置 > oozie-site.xml 的 Oozie Server 高级配置代码段（安全阀）
2. 新增如下配置：

```
<property>  
  <name>oozie.launcher.fs.hdfs.impl.disable.cache</name>
  <value>true</value>
</property>  
<property>  
  <name>oozie.action.max.output.data</name>
  <value>5000000</value>
</property>  
<property>  
  <name>oozie.processing.timezone</name>
  <value>GMT+0800</value>
</property>  
<property>  
  <name>oozie.service.ELService.ext.functions.workflow</name>
  <value>horus:getDateFromNow=io.terminus.horus.oozie.udf.TerminusCoordELFunctions#getDateFromNow</value>
</property>
```

- 时区的配置和自定义function等配置，根据自己的需求而定。
- 非CDH平台直接修改oozie的 `oozie-site.xml` 配置即可

> 3、 上传自定义的 `share jar` 除了oozie自身的sharelib之外，往往我们还需上传自己的公用jar包

```
1. 根据我们平台自己的需求，我们将jar包存放在HDFS的 /user/oozie/share/custom 的目录下  
```

![img](http://7xsz2j.com1.z0.glb.clouddn.com/2016-11-25-045843.jpg)

> 4、删除一些oozie share lib

```
删除oozie的sharelib的spark目录下的几个jar，如下
* javax.servlet-3.0.0.v201112011016.jar
* guava-14.0.1.jar
```

> 5、扩展平台功能

```
详情参考说明：
sqoop: http://git.terminus.io/bigdata/sqoop  
oozie: http://git.terminus.io/bigdata/oozie  
```

> 6、最后重启oozie服务

# 2. hive 相关配置

> 1、上传hive-site.xml

```
这个是horus平台需要：
1. 将 /etc/hive/conf/hive-site.xml 文件上传到hdfs的一个目录  
2. horus平台项目的启动配置加上如下配置（Xxxx就是上传的hdfs路径）  
    -Doozie.jobXml=Xxxx
```

> 2、设置hive的辅助 JAR

```
1. CDH平台前往 hive > 配置 > Hive 辅助 JAR 目录  
2. 输入 /var/lib/hive  
3. 将一些辅助jar，如UDF之类的jar加入其中（比如平台的 woothee.jar）  
4. 重启服务  
```

- 非cdh平台直接在hive-site.xml文件中配置`hive.aux.jars.path`即可。
- 注意所有安装hive的机器都要加入这些jar

> 3、打开hive动态分区(视需求而定)

```
1. CDH平台前往 hive > 配置 > hive-site.xml 的 HiveServer2 高级配置代码段（安全阀）  
2. 加入配置如下  
    <property>
        <name>hive.exec.dynamic.partition</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.exec.dynamic.partition.mode</name>
        <value>nonstrict</value>
    </property>
    <property>
        <name>hive.exec.max.dynamic.partitions</name>
        <value>10000</value>
    </property>
    <property>
        <name>hive.exec.max.dynamic.partitions.pernode</name>
        <value>10000</value>
    </property>
3. 重启hive服务  
```

- 非cdh平台修改`hive-site.xml`即可

> 4、小文件合并

1. CDH平台前往 hive > 配置 > 启用小文件合并 - Map-Reduce 作业 hive.merge.mapredfiles > 勾上后面的选项即可

> 5、解决Hive元数据乱码问题

```
* 安装默认的编码是latin1, 修改元数据表结构, 不要修改整库的编码。
use hive;  
alter table COLUMNS_V2 modify column COMMENT varchar(256) character set utf8;  
alter table TABLE_PARAMS modify column PARAM_VALUE varchar(4000) character set utf8;  
alter table PARTITION_PARAMS  modify column PARAM_VALUE varchar(4000) character set utf8;  
alter table PARTITION_KEYS  modify column PKEY_COMMENT varchar(4000) character set utf8;  
alter table INDEX_PARAMS  modify column PARAM_VALUE  varchar(4000) character set utf8;  
```

- 注意：`非CDH Manager安装`需要修改连接mysql的 `url`，并重启hive服务

```
jdbc:mysql://YOUR_HOST/hivemeta?createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8  
```