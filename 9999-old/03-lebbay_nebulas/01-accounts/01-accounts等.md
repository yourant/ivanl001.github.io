[toc]



#### 1, ip, host和证书

```shell
# 证书见文件夹目录
# ip和host如下表
```



* 新的

| jumper-name     | pub-IP                       | priv-IP       | ssh-port | username | hostname |
| --------------- | ---------------------------- | ------------- | -------- | -------- | -------- |
| bi-prod-master1 | 54.145.185.81                | 172.31.14.204 | 38022    | centos   | nebula01 |
| bi-prod-master2 | 3.82.10.149                  | 172.31.0.112  | 38022    | centos   | nebula02 |
| bi-prod-slave1  | 3.88.25.105                  | 172.31.8.64   | 38022    | centos   | nebula03 |
| bi-prod-slave2  | 54.159.76.106                | 172.31.15.12  | 38022    | centos   | nebula04 |
| bi-prod-slave3  | 18.212.157.127(宕机一次变过) | 172.31.10.15  | 38022    | centos   | nebula05 |
| bi-prod-slave4  | 3.221.134.225                | 172.31.14.123 | 38022    | centos   | nebula06 |
| bi-prod-slave5  | 54.80.154.165                | 172.31.2.220  | 38022    | centos   | nebula07 |
|                 | 18.234.30.245                | 172.31.11.207 | 38022    | centos   | nebula08 |
|                 | 3.87.170.143                 | 172.31.14.139 | 38022    | centos   | nebula09 |
|                 | 54.173.16.143                | 172.31.6.191  | 38022    | centos   | nebula10 |

```shell
# bi-prod-slave3 2020-07-08 晚上九点10点左右宕机过一次，不影响集群使用。2020-07-09 早上10点左右重启，内网不变，外网发生变化，未影响
# 机器down机一次，未记录时间，未影响
# 机器未down机，但是失去心跳，无法连上。未记录时间。ha生效，未影响
# nebula05磁盘出现问题导致写入缓慢，cpu io等待时间长。更换磁盘后恢复。未影响数据
```



172.31.8.64:9092, 172.31.15.12:9092,172.31.10.15:9092,172.31.14.123:9092,172.31.2.220:9092

nebula03:9092,nebula04:9092,nebula05:9092,nebula06:9092,nebula07:9092

```
nebula03:9092,nebula04:9092,nebula05:9092,nebula06:9092,nebula07:9092
```



* 已过期

| jumper-name     | pub-IP        | priv-IP       | ssh-port | username | hostname |
| --------------- | ------------- | ------------- | -------- | -------- | -------- |
| bi-prod-master1 | 3.80.215.22   | 172.31.29.76  | 38022    | centos   | nebula01 |
| bi-prod-master2 | 3.214.144.104 | 172.31.70.190 | 38022    | centos   | nebula02 |
| bi-prod-slave1  | 3.81.55.28    | 172.31.12.253 | 38022    | centos   | nebula03 |
| bi-prod-slave2  | 3.83.125.16   | 172.31.34.134 | 38022    | centos   | nebula04 |
| bi-prod-slave3  | 18.205.67.227 | 172.31.85.249 | 38022    | centos   | nebula05 |
| bi-prod-slave4  | 3.221.134.225 | 172.31.19.140 | 38022    | centos   | nebula06 |
| bi-prod-slave5  | 3.230.171.154 | 172.31.77.55  | 38022    | centos   | nebula07 |



```shell
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6


172.31.14.204  nebula01
172.31.0.112   nebula02
172.31.8.64    nebula03
172.31.15.12   nebula04
172.31.10.15   nebula05
172.31.14.123  nebula06
172.31.2.220   nebula07
```



#### 2, nebula01, nebula02上mysql的账号密码

* 数据库中大数据中的用户的用户名和账号密码暂时不记
* 该数据库是自己安装，账号密码仅本人有

```shell
# nebula01-mysql
3.80.215.22
# 内网
172.31.14.204
3306
root
yfrmJ1Vt9_NyCfi7

# nebula02-mysql是nebula01上mysql的binlog备份数据库
3.214.144.104
3306
root
yfrmJ1Vt9_NyCfi7


# 给上面两个数据库创建只读用户
create user 'nebulas'@'%' identified by '8Bvc9YUB_AvPyQIS';
grant select on *.* to 'nebulas'@'%';
show grants;
show grants for 'nebulas'@'%';
# 两台数据库等只读权限数据库账号密码
ip
3306
nebulas
8Bvc9YUB_AvPyQIS

```



#### 3, jumpServer账号密码

* 密码为下发密码，暂时没有更改(20200605更改)

```shell
http://jmp.opsfun.com:8082/
dfzhang
# dfzhang@123
# 更改成新密码
lebbay_dfzhang
```



#### 4, cm相关账号密码

```shell

```



#### Hue账号密码

```shell
http://cdh.opsfun.com:8889/
hdfs
hdfs01
```







#### 5, superset相关账号密码

```shell
http://cdh2.opsfun.com:8089/login/
admin
lebbay_admin

http://cdh2.opsfun.com:8089/login/
bool.zhang
dfjk


http://cdh.opsfun.com:8089/login/
admin
lebbay_admin01
```



#### 6, canal消费zazaie的binlog账号密码

```shell
# binlog连接主库
azaziedb.cbb0nles4v8i.us-east-1.rds.amazonaws.com
3306
canal
afA!^v$n8eicEA6C
```



#### 7, rubix数据库的账号密码

```shell
adsdb.cuqmct6whng1.us-east-1.rds.amazonaws.com
3306
bigdata
jCjq7K0CLkHJjeAn
```



#### 8, report报表数据库账号密码

```mysql
# 数据库管理员权限
地址：report-system.cbb0nles4v8i.us-east-1.rds.amazonaws.com  
端口：3306
用户名：admin
密码：Jde27dePlWbx32aMTr

# 这个给叶韬写入页面评分数据方便放在bi报表系统上
地址：report-system.cbb0nles4v8i.us-east-1.rds.amazonaws.com  
端口：3306
用户名：yetao
密码：yetao_azazie_20200228

# 这个是给ai那边拉取结果表app_review
地址：report-system.cbb0nles4v8i.us-east-1.rds.amazonaws.com  
端口：3306
用户名：ai_query
密码：z8CGb2DvtXkQsNkc
```



#### 9, azazie数据库

```mysql
# 杰哥给的
azaziedbslave.cbb0nles4v8i.us-east-1.rds.amazonaws.com
aztech1
8m4UquCBkNYn
```



#### 10, hosts添加内容

* 新的

```shell
# 内网hosts，用在内网内的系统互相关联
172.31.14.204  nebula01
172.31.0.112   nebula02
172.31.8.64    nebula03
172.31.15.12   nebula04
172.31.10.15   nebula05
172.31.14.123  nebula06
172.31.2.220   nebula07

# 外网hosts,一般用在本机上加远程服务器
54.145.185.81  nebula01
3.82.10.149    nebula02
3.88.25.105    nebula03
54.159.76.106  nebula04
54.146.107.217 nebula05
3.221.134.225  nebula06
54.80.154.165  nebula07
```

* 如下内容已经过期

```shell
# 内网hosts，用在内网内的系统互相关联
172.31.29.76  nebula01
172.31.70.190 nebula02
172.31.12.253 nebula03
172.31.34.134 nebula04
172.31.85.249 nebula05
172.31.19.140 nebula06
172.31.77.55  nebula07

# 外网hosts,一般用在本机上加远程服务器
3.80.215.22    nebula01
3.214.144.104  nebula02
3.81.55.28     nebula03
3.83.125.16    nebula04
18.205.67.227  nebula05
18.206.174.196 nebula06
```



