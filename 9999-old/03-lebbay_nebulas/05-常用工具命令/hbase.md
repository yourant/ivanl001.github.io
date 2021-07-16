```shell
# 启动queryServer方便redash或者superset连接
nohup /opt/cloudera/parcels/APACHE_PHOENIX/lib/phoenix/bin/queryserver.py > /data/phoenix/queryserver.log 2>&1 &



#  重客户端
phoenixHbase
jdbc:phoenix:nebula03:2181
org.apache.phoenix.jdbc.PhoenixDriver



nohup java -jar gmall0105-logger-0.0.1-SNAPSHOT.jar > /usr/local/mock/gmall0105-logger/target/gmall0105-logger.log 2>&1 &



#  轻客户端

类名：org.apache.phoenix.queryserver.client.Driver
模版：jdbc.phoenix:thin:url=http://{host}:{port};serialization=PROTOBUF
默认端口：8765
驱动jar：org.apache.phoenix:phoenix-queryserver-client:5.0.0-HBase-2.0
<!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-queryserver-client -->
<dependency>
    <groupId>org.apache.phoenix</groupId>
    <artifactId>phoenix-queryserver-client</artifactId>
    <version>5.0.0-HBase-2.0</version>
</dependency>





```





```sql
-- phoenix 联合主键
create table if not exists learn.us_population(state char(2) not null, city varchar not null, population bigint 
                                        constraint my_pk primary key (state, city));
 
--  插入数据
UPSERT INTO LEARN.US_POPULATION values ('NY', 'new york', 1000);
 

-- 通过hbase shell直接查看该表的原始数据，可以发现key 是：NYnew york
```



```sql
----------映射表好像查不了非主键外的其他字段

-- hbase表和phoenix表的相互映射
-- 先在hbase shell里面建表
create 'LEARN.FRUIT', 'info'

put 'LEARN.FRUIT','1001','info:name','apple'
put 'LEARN.FRUIT','1001','info:color','red'
put 'LEARN.FRUIT','1002','info:name','orange'
put 'LEARN.FRUIT','1002','info:color','blue'


-- 再phoenix中进行映射
-- 1, view映射（view里面不能修改数据，只能查看数据）
create view "LEARN"."FRUIT" (id varchar primary key, "info"."name" varchar, "info"."color" varchar);

-- 2, table映射
create table "LEARN"."FRUIT" (id varchar primary key, "info"."name" varchar, "info"."color" varchar) column_encoded_bytes=0;

UPSERT INTO LEARN.FRUIT values ('1003', 'blueberry', 'blue');

```





```xml
// maven依赖
<!-- https://mvnrepository.com/artifact/org.apache.phoenix/phoenix-queryserver-client -->
<dependency>
  <groupId>org.apache.phoenix</groupId>
  <artifactId>phoenix-queryserver-client</artifactId>
  <version>5.0.0-HBase-2.0</version>
</dependency>
```



```java
package com.azazie.report.controller.test;

import org.apache.phoenix.queryserver.client.ThinClientUtil;

import java.sql.*;

/**
 * @author : 不二
 * @date : 2021/3/18-下午8:52
 * @desc :
 **/
public class ABC {

    public static void main(String[] args) throws SQLException {
        String nebulas02 = ThinClientUtil.getConnectionUrl("nebula02", 8765);
        System.out.println(nebulas02);

        Connection connection = DriverManager.getConnection(nebulas02);
        PreparedStatement preparedStatement = connection.prepareStatement("select * from learn.fruit");

        ResultSet resultSet = preparedStatement.executeQuery();
        while (resultSet.next()) {
            System.out.println(
                    resultSet.getString(1) + "\t" +
                    resultSet.getString(2) + "\t" +
                    resultSet.getString(3));
        }
    }
}
```





```mysql

-- hive中操作会直接在hbase中自动创建
-- 把hbase和hive表映射起来
CREATE TABLE hbase.employee(
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
comm double,
deptno int)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,info:ename,info:job,info:mgr,info:hiredate,info:sal,info:comm,info:deptno") 
TBLPROPERTIES ("hbase.table.name" = "LEARN.EMPLOYEE");


-- 这个是用来测试的hive表
CREATE TABLE hbase.employee_tmp(
empno int,
ename string,
job string,
mgr int,
hiredate string,
sal double,
comm double,
deptno int)
row format delimited 
fields terminated by '\001'
lines terminated by '\n'

-- 然后hbase.employee_tmp中造一些数据，然后插入到hbasse表中
insert into employee
select * from employee_tmp as a;

-- 最后在hbase中能查到相关数据了



-- 然后在phoenix中创建个view进行查询
create view "LEARN"."EMPLOYEE" (
empno varchar primary key,
"info"."ename" varchar,
"info"."job" varchar,
"info"."mgr" varchar,
"info"."hiredate" varchar,
"info"."sal" varchar,
"info"."comm" varchar,
"info"."deptno" varchar);


drop view learn.employee;

-- 如果想要编辑，也可以创建table
create table "LEARN"."EMPLOYEE" (
 empno varchar primary key,
"info"."ename" varchar,
"info"."job" varchar,
"info"."mgr" varchar,
"info"."hiredate" varchar,
"info"."sal" varchar,
"info"."comm" varchar,
"info"."deptno" varchar) column_encoded_bytes=0;
-- 一定要加最后一行


upsert into learn.employee values ('100100', 'ivanl100100', 'service', '100100', '1990-03-05', '333.9','332.4','25');

-- 小写字母需要添加引号！！！
select "deptno", count(1) as sal from learn.employee as e group by "deptno";




select "color", count(1) from learn.fruit group by "color"
```

