[TOC]

## 1, mariadb基于binlog主从备份

### 00, binlog的三种模式

* statement: 记录master节点语句，在slave运行，如果是随机数，则可能主从不一样
* row：直接记录行数据，虽然数据可以保持完全一致，但是网络IO可能会比较大
* mix：一般情况下会使用statement模式，特殊情况下使用row模式 



### 01,  配置主节点

```shell
vim /etc/my.cnf.d/server.cnf

# -------------------------------------主服务器：-------------------------------------
# server-id 必须唯一
server-id=1
#开启二进制日志
log-bin=/var/lib/mysql/binlog/mysql-binlog
log-bin-index=/var/lib/mysql/binlog/mysql-binlog.index
# binlog的行模式
binlog_format=row
# 只记录testdb库变化,多个库用‘,’分隔
# binlog-do-db = testdb
#忽略mysql库变化，多个库用‘,’分隔
# binlog-ignore-db=mysql
```

### 02, 主节点创建需要的文件夹

```shell
# 创建binlog目录
mkdir /var/lib/mysql/binlog/
chown -R mysql.mysql /var/lib/mysql/binlog/
```



### 03,  重启主服务器的mysql服务;

```shell
# 重启主服务器的mysql服务;
systemctl restart mariadb.service

# 验证：
# 验证binlog模式是什么模式，我这里设置的是row模式
show variables like 'binlog_format';
# 查看binlog是否开启，我这里显示on，开启
show variables like 'log_bin';
```



### 04, mysql命令行中给从节点赋予权限

```shell
# 先进入mysql
mysql -u root -p

# 给从节点103赋予权限
grant replication slave on *.* to 'root'@'192.168.215.103' identified by ',.';
flush privileges;


grant replication slave on *.* to 'root'@'192.168.147.104' identified by 'Zhangdf_123';
flush privileges;
```



### 05, 查看binlog文件和位置

```shell
# 查看主节点的binlog文件和位置
# 这里锁表的目的是为了防止位移发生变化
flush tables with read lock;
show master status;
UNLOCK TABLES;
```



### 06, 配置从节点

```shell
vim /etc/my.cnf.d/server.cnf

# -------------------------------------从服务器：-------------------------------------
vim /etc/my.cnf.d/server.cnf
# server-id 必须唯一
server-id=10
# 开启二进制日志
relay-log=/var/lib/mysql/relaylog/mysql-relaylog
relay-log-index=/var/lib/mysql/relaylog/relay-log.index
read-only=ON
# 当做级联复制，或者从库做备份时A-->B-->C，B服务需要开启log-bin和log-slave-updates，二者缺一不可
#log-slave-updates
# 当binlog日志较多时，此参数的值意思是只保留7天内的数据。主库及从库都可以配置此参数。
# expire_logs_days =7
# 只同步testdb库，多个库用‘,’分隔
# replication-do-db = testdb
# 不同步mysql库，多个库用‘,’分隔
# replication-ignore-db=mysql
# 只同步test_tb表，多个表用‘,’分隔
# replication-do-table = test_tb
# 不同步test_tb表，多个表用‘,’分隔
# replication-ignore-table = test_tb
# 同replication-do-table一样，可以加通配符
# replication-wild-do-table = test_tb
# 同replication-ignore-table一样，可以加通配符
# replication-wild-ignore-table = test_tb
```



### 07, 从节点创建需要的文件夹

```shell
# 创建binlog目录
mkdir /var/lib/mysql/relaylog/
chown -R mysql.mysql /var/lib/mysql/relaylog/
```



### 08, 重启从节点

```shell
# 重启从服务器的mysql服务;
systemctl restart mariadb.service
```



### 09, mysql命令行中设置并启动从节点

```shell
# master_log_file是刚才show master status时候的那个文件或者你想要启动启动的那个文件
change master to master_host='192.168.215.101',master_user='root',master_password=',.',master_log_file='mysql-binlog.000014',master_log_pos=245;

#启动从服务器复制功能
start slave;

# 验证从节点的状态
show slave status \G;

# 只有下面两个属性全部为Yes的时候才算成功
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```



## 2，报错异常解决

### 01, Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'

> Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'



- 参考资料：
  - https://www.cnblogs.com/charles1ee/p/6364448.html
  - https://blog.csdn.net/Linux_newbie_rookie/article/details/77568552



- 1, 从库先停止slave

```shell
stop slave;
```



- 2，主库刷写更新binlog文件：

```shell
flush logs;
```



- 3，查询主库状态：

```shell
# 根据文件和位置，在后面设置从库的binlog文件和位置
show master status \G;
```



- 4,  到从库上进行设置：

```shell
change master to master_host='192.168.215.101',master_user='root',master_password=',.',master_log_file='mysql-binlog.000016',master_log_pos=245;
```



- 5，从库启动： 

```shell
start slave;
```



- 6，查看从库状态：

```shell
show slave status \G;


# 如果下面两个属性全部为Yes的时候就ok了
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```







## 3，草稿

 ```shell
vim /etc/my.cnf.d/server.cnf

# -------------------------------------主服务器：-------------------------------------
# server-id 必须唯一
server-id=1
#开启二进制日志
log-bin=/var/lib/mysql/binlog/mysql-binlog
log-bin-index=/var/lib/mysql/binlog/mysql-binlog.index
# binlog的行模式
binlog_format=row
# binlog-do-db = testdb //只记录testdb库变化,多个库用‘,’分隔
# binlog-ignore-db=mysql //忽略mysql库变化，多个库用‘,’分隔



# 创建binlog目录
mkdir /var/lib/mysql/binlog/
chown -R mysql.mysql /var/lib/mysql/binlog/
# 重启主服务器的mysql服务;
systemctl restart mariadb.service
#  给从节点赋予权限
grant replication slave on *.* to 'root'@'192.168.215.%' identified by ',.';
flush privileges;
# 这里锁表的目的是为了防止位移发生变化
flush tables with read lock;
show master status;

UNLOCK TABLES;

# -------------------------------------从服务器：-------------------------------------
vim /etc/my.cnf.d/server.cnf
# server-id 必须唯一
server-id=10
# 开启二进制日志
relay-log=/var/lib/mysql/relaylog/mysql-relaylog
relay-log-index=/var/lib/mysql/relaylog/relay-log.index
read-only=ON
# 当做级联复制，或者从库做备份时A-->B-->C，B服务需要开启log-bin和log-slave-updates，二者缺一不可
#log-slave-updates
# 当binlog日志较多时，此参数的值意思是只保留7天内的数据。主库及从库都可以配置此参数。
# expire_logs_days =7
# 只同步testdb库，多个库用‘,’分隔
# replication-do-db = testdb
# 不同步mysql库，多个库用‘,’分隔
# replication-ignore-db=mysql
# 只同步test_tb表，多个表用‘,’分隔
# replication-do-table = test_tb
# 不同步test_tb表，多个表用‘,’分隔
# replication-ignore-table = test_tb
# 同replication-do-table一样，可以加通配符
# replication-wild-do-table = test_tb
# 同replication-ignore-table一样，可以加通配符
# replication-wild-ignore-table = test_tb

# 创建binlog目录
mkdir /var/lib/mysql/relaylog/
chown -R mysql.mysql /var/lib/mysql/relaylog/

# 重启主服务器的mysql服务;
systemctl restart mariadb.service

# master_log_file是刚才show master status时候的那个文件或者你想要启动启动的那个文件
mysql> change master to master_host='192.168.215.101',master_user='root',master_password=',.',master_log_file=' mysql-binlog.000001',master_log_pos=245;
mysql> start slave;    #启动从服务器复制功能
mysql> show slave status\G



change master to master_host='192.168.215.101',master_user='root',master_password=',.',master_log_file='mysql-binlog.000006',master_log_pos=245;
 ```



* > mysql从库Last_IO_Error: Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'报错处理

*　https://www.cnblogs.com/charles1ee/p/6364448.html
*　1, 从库先停止：stop slave;
*　2，主库刷新：flush logs;
*　3，查询主库状态：show master status\G;
*　4,  到从库上进行设置：change master to master_host='192.168.215.101',master_user='root',master_password=',.',master_log_file='mysql-binlog.000006',master_log_pos=245;
*　5，从库启动： start slave;
*　6，查看从库状态：

