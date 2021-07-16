



[toc]



show processlist;

show full processlist;

查看负载

uptime









## 1, 硬件优化

### 1.1, cpu

8-16 core

### 1.2, 内存

96-128G

3-4个实例

### 1.3, disk

数量越多越好

性能: ssd(高并发业务) > sas(普通业务) > sata(线下业务)

   

## 2, my.conf里参数的优化

优化空间比较小

```properties
# 这个建议配置最大到物理内存的30%-50%
innodb_buffer_pool_size=2048M
```

命令监控：show global status \G;

mysql调优工具：mysqlreport等



## 3, sql语句的优化

### 3.1, 索引优化

#### 慢sql

慢查询日志分析工具：mysqldumpslow, myprofi, mysqlsla(最好)



3.2, 





4, 架构的优化

5, 流程，制度，安全优化

