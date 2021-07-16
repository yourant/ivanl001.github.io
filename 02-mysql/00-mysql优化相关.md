[toc]

<extoc></extoc>



## 0, mysql连接命令

```mysql
-- mysql -h hostname -P port -u username -P
-- 输入密码

mysql -h report-system.cbb0nles4v8i.us-east-1.rds.amazonaws.com -P 3306 -u admin -p 
Jde27dePlWbx32aMTr


mysql -h azaziedbslave.cbb0nles4v8i.us-east-1.rds.amazonaws.com -P 3306 -u aztech1 -p
8m4UquCBkNYn
```



## 1, 查看数据库磁盘占用

```mysql
# 1，查看所有库的数据占用磁盘大小
select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024/1024,2),' GB') as data_size,
concat(truncate(sum(index_length)/1024/1024/1024,2),'GB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
order by data_length desc;

# 2，查看制定的数据库每张表占用磁盘大小
select TABLE_NAME as table_name, concat(truncate(data_length/1024/1024,2),' MB') as data_size,
concat(truncate(index_length/1024/1024,2),' MB') as index_size
from information_schema.tables 
where TABLE_SCHEMA = 'azazie'
group by TABLE_NAME
order by data_length desc
limit 100;

```



## 2, 进程监控及慢查询

```mysql
-- mysql连接状态查看
show PROCESSLIST;
show full PROCESSLIST;


-- 这个命令是？
show PROFILES;
show PROFILE for QUERY 54;


-- 1.查看慢查询是否开启以及慢查询日志存储位置
show variables like 'slow_query%';
-- 2.查看慢查询的时间
show variables like 'long_query_time';
-- 3.开启慢查询日志，方便后续进行检查
set global slow_query_log='ON';
show variables like '%query%';


-- 进行测试
select sleep(20);


-- 根据show variables like 'slow_query%';可以查到日志文件的位置
-- 根据位置即可找到慢日志
-- 然后如果想要格式化日志文件，可以使用下面的命令：mysqldumpslow


-- s是排序，sort的意思，c是count，-s c 也就是按照请求个数排序
-- t是top，-t 10也就是按照排序取前10个
mysqldumpslow -s c -t 10 /usr/local/mysql/data/mac-slow.log
-- s是排序，sort的意思，t是time，-s t 也就是按照请求时间排序
mysqldumpslow -s t -t 10 /usr/local/mysql/data/mac-slow.log

```

