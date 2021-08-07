```shell
CREATE FUNCTION functionname AS 'com.lijie.test.udf.GetId' using jar 'hdfs://master:8020/user/hive/udf/udftest.jar.jar';
```







```shell
##### !!!  因为在hive配置中配置了Hive 辅助 JAR 目录 为： /home/hive/lib，所以后续需要把这个目录给配置
add jar /opt/cloudera/parcels/CDH/lib/hive/lib/hiveCustomFunction-0.0.1-SNAPSHOT.jar

# create temporary function page_mapping as 'com.lebbay.hive.udf.HivePageMapping';
# create function page_mapping as 'com.lebbay.hive.udf.HivePageMapping';


create function page_mapping as 'com.lebbay.hive.udf.HivePageMapping'



# 第一步 上传jar包到hdfs
hdfs dfs -put /opt/cloudera/parcels/CDH/lib/hive/lib/hiveCustomFunction-0.0.1-SNAPSHOT.jar /user/hive/func
# 第二步 永久添加udf的方法
CREATE FUNCTION page_mapping as 'com.lebbay.hive.udf.HivePageMapping' using JAR "hdfs://nameservice1/user/hive/func/hiveCustomFunction-0.0.1-SNAPSHOT.jar";

CREATE FUNCTION platform_mapping as 'com.lebbay.hive.udf.HivePlatformMapping' using JAR "hdfs://nameservice1/user/hive/func/hiveCustomFunction-0.0.1-SNAPSHOT.jar";




CREATE FUNCTION ip_2_region as 'com.lebbay.hive.udf.IP2Region'



https://repo1.maven.org/maven2/org/apache/hive/hive-contrib/1.1.0/hive-contrib-1.1.0.jar


select page_mapping(4,4,'/checkout_order_success');

select page_mapping(2,1,'/user/login');
select page_mapping(5,1,'/user/login');
select page_mapping(4,1,'/user/login');

select page_mapping(2,1,'/user');



select page_mapping(5,3,'home');


CREATE FUNCTION platform_mapping as 'com.lebbay.hive.udf.HivePlatformMapping';



CREATE FUNCTION page_mapping as 'com.lebbay.hive.udf.HivePageMapping';
CREATE FUNCTION ip_2_region_udf as 'com.lebbay.hive.udf.IP2RegionUDF';
CREATE FUNCTION ip_2_region_udtf as 'com.lebbay.hive.udtf.IP2RegionUDTF';

drop function platform_mapping;
drop function page_mapping;
drop function ip_2_region_udf;
drop function ip_2_region_udtf;

```

```ruby
hive> create function growth.correct_resolution as 'correct_resolution.CorrectResolution' using JAR "hdfs://kingkong/usr/xx/xxx/udf/CorrectResolution.jar";
converting to local hdfs://kingkong/usr/xx/xxx/udf/CorrectResolution.jar
Added [/tmp/xx-xx-xx-xx-xx_resources/CorrectResolution.jar] to class path
Added resources: [hdfs://kingkong/usr/xx/xxx/udf/CorrectResolution.jar]
OK
Time taken: 0.058 seconds
hive> select growth.correct_resolution("768*1280"), 'xxx' as t;
OK
1280*768    xxx
```

