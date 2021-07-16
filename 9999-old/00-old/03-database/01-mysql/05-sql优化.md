```mysql
-- mysql连接状态查看
show PROCESSLIST

show PROFILES;
show PROFILE for QUERY 54;



-- 1.查看慢查询相关参数
show variables like 'slow_query%';

-- 查看慢查询的时间
show variables like 'long_query_time';

-- 开启慢查询日志，方便后续进行检查
set global slow_query_log='ON';

show variables like '%query%';

select sleep(2);


mysqldumpslow -s c -t 10 /rdsdbdata/log/slowquery/mysql-slowquery.log

mysqldumpslow -s r -t 10 /database/mysql/mysql06_slow.log


mysqldumpslow --help

select * from slowquery/mysql-slowquery.log


show global status like '%slow%';


select * from mysql.slow_log limit 10 




```



