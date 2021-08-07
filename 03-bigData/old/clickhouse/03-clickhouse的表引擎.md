[toc]



```shell
# 表引擎的使用方式就是必须显形在创建表时定义该表使用的引擎，以及引擎使用的相关参数。如：
# 特别注意：引擎的名称大小写敏感
create table t_tinylog (id String, name String) engine=TinyLog;
insert into t_memory values('1', 'ivanl001');

```





### ***5.2  TinyLog***

```sql
create table t_tinylog (id String, name String) engine=TinyLog;
insert into t_tinylog values('1', 'ivanl001');
```



  以列文件的形式保存在磁盘上，`不支持索引`，没有并发控制。一般保存少量数据的小表，生产环境上作用有限。可以用于平时练习测试用。



### ***5.3***  ***Memory***

```sql
create table t_memory ( id String, name String) engine=Memory;

insert into t_memory values('1', 'ivanl001');
```



内存引擎，数据以未压缩的原始形式直接保存在内存当中，服务器重启数据就会消失。读写操作不会相互阻塞，不支持索引。简单查询下有非常非常高的性能表现（超过10G/s）。

一般用到它的地方不多，除了用来测试，就是在需要非常高的性能，同时数据量又不太大（上限大概 1 亿行）的场景。



### ***5.4 MergeTree***

```sql
# 合并树
create table t_order_mt(
id UInt32,
sku_id String,
total_amount Decimal(16,2),
create_time  Datetime
) engine =MergeTree
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id)

insert into  t_order_mt
values(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00')
(102,'sku_002',12000.00,'2020-06-01 13:00:00')
(102,'sku_002',600.00,'2020-06-02 12:00:00')

# 如果多次插入的话， 即便是应该是在相同分区的数据，也可能会被临时创建新的分区，后台会定时的合并分区
# 如果想要手动合并分区
optimize table t_order_mt final;

```



Clickhouse 中最强大的表引擎当属 MergeTree （合并树）引擎及该系列（*MergeTree）中的其他引擎。地位可以相当于innodb之于Mysql。 而且基于MergeTree，还衍生除了很多小弟，也是非常有特色的引擎。

建表语句

 MergeTree其实还有很多参数(绝大多数用默认值即可)，但是三个参数是更加重要的，也涉及了关于MergeTree的很多概念。



#### ***5.4.1*** ***p******artition by*** ***分区 （可选项）***

***作用：*** 学过hive的应该都不陌生，分区的目的主要是降低扫描的范围，优化查询速度。

***如果不填：*** 只会使用一个分区。

  ***分区目录：*** MergeTree 是以列文件+索引文件+表定义文件组成的，但是如果设定了分区那么这些文件就会保存到不同的分区目录中。

  ***并行：***分区后，面对涉及跨分区的查询统计，clickhouse会以分区为单位并行处理。

  

  ***数据写入与分区合并：***

  任何一个批次的数据写入都会产生一个临时分区，不会纳入任何一个已有的分区。写入后的某个时刻（大概10-15分钟后），clickhouse会自动执行合并操作（等不及也可以手动通过optimize执行），把临时分区的数据，合并到已有分区中。

optimize table xxxx [final]



#### ***5.4.2 primary key******主键(可选)***

clickhouse中的主键，和其他数据库不太一样，***它只提供了数据的一级索引，但是却不是唯一约束***。这就意味着是可以存在相同primary key的数据的。

主键的设定主要依据是查询语句中的where 条件。

根据条件通过对主键进行某种形式的二分查找，能够定位到对应的index granularity,避免了全包扫描。

index granularity： 直接翻译的话就是索引粒度，指在稀疏索引中两个相邻索引对应数据的间隔。clickhouse中的MergeTree默认是8192。

官方不建议修改这个值，除非该列存在大量重复值，比如在一个分区中几万行才有一个不同数据。

稀疏索引：

![img](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/wpsEcVTOn.jpg) 

稀疏索引的好处就是可以用很少的索引数据，定位更多的数据，代价就是只能定位到索引粒度的第一行，然后再进行进行一点扫描。



#### ***5.4.3 order by*** ***（必选）***

order by 设定了***分区内***的数据按照哪些字段顺序进行有序保存。

order by是MergeTree中唯一一个必填项，甚至比primary key 还重要，因为当用户不设置主键的情况，很多处理会依照order by的字段进行处理（比如后面会讲的去重和汇总）。

***要求：主键必须是o******rder by******字段的前缀字段。***

比如order by 字段是 (id,sku_id)  那么主键必须是id 或者(id,sku_id)





#### ***5.4.4*** ***二级索引***

 目前在clickhouse的官网上二级索引的功能是被标注为***实验性***的。

所以使用二级索引前需要增加设置

```sql
# 在高的版本里面好像不需要设置了
# set allow_experimental_data_skipping_indices=1;

create table t_order_mt2(
id UInt32,
sku_id String,
total_amount Decimal(16,2),
create_time  Datetime,
-- 二级索引设置
INDEX a total_amount TYPE minmax GRANULARITY 5
) engine =MergeTree
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);


insert into  t_order_mt2
values(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00')
(102,'sku_002',12000.00,'2020-06-01 13:00:00')
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

其中***GRANULARITY N*** 是设定二级索引对于一级索引粒度的粒度。 

```sql
-- 无索引表
clickhouse-client --send_logs_level=trace <<< 'select * from t_order_mt  where total_amount > 1000'

-- 有索引表
clickhouse-client --send_logs_level=trace <<< 'select * from t_order_mt2  where total_amount > 1000'
```



#### ***5******.4.5*** ***数据TTL*** 

TTL即Time To Live，MergeTree提供了可以管理数据或者列的生命周期的功能。 

* 列级别TTL

```sql
 optimize table t_order_mt3 final;
 
 # 这种没验证出来， 手动optimize也不行
 create table t_order_mt3(
   id UInt32,
   sku_id String,
   total_amount Decimal(16,2) TTL create_time+interval 10 SECOND,
   create_time  Datetime 
 ) engine =MergeTree
 partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id, sku_id);
  
  
# 插入数据
insert into  t_order_mt3
values(106,'sku_001',1000.00,'2021-07-09 20:42:52')
```



* 表级TTL 

针对整张表  

```sql
 create table t_order_mt3_v1(
   id UInt32,
   sku_id String,
   total_amount Decimal(16,2),
   create_time  Datetime 
 ) engine =MergeTree
 TTL create_time + INTERVAL 10 SECOND
 partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id, sku_id);

# 下面的这条语句是数据会在create_time 之后10秒丢失
# alter table t_order_mt3 MODIFY TTL create_time + INTERVAL 10 SECOND;

insert into  t_order_mt3_v1
values(106,'sku_001',1000.00,'2021-07-09 20:53:40');
```



涉及判断的字段必须是Date或者Datetime类型，推荐使用分区的日期字段。

能够使用的时间周期：

\- SECOND

\- MINUTE

\- HOUR

\- DAY

\- WEEK

\- MONTH

\- QUARTER

\- YEAR 





### ***5.5  Replacing******M******ergeTree***

ReplacingMergeTree是MergeTree的一个变种，它存储特性完全继承MergeTree，只是多了一个去重的功能。 

尽管MergeTree可以设置主键，但是primary key其实没有唯一约束的功能。如果你想处理掉重复的数据，可以借助这个ReplacingMergeTree。

***去重时机***：数据的去重只会在合并的过程中出现。合并会在未知的时间在后台进行，所以你无法预先作出计划。有一些数据可能仍未被处理。

***去重范围***：如果表经过了分区，去重只会在分区内部进行去重，不能执行跨分区的去重。

 

所以ReplacingMergeTree能力有限， ReplacingMergeTree 适用于在后台清除重复的数据以节省空间，但是它不保证没有重复的数据出现。

```sql
create table t_order_rmt(
id UInt32,
sku_id String,
total_amount Decimal(16,2) ,
create_time  Datetime 
) engine =ReplacingMergeTree(create_time)
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);



insert into  t_order_rmt
values(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00')
(102,'sku_002',12000.00,'2020-06-01 13:00:00')
(102,'sku_002',600.00,'2020-06-02 12:00:00')
```



通过测试得到结论：

Ø 实际上是使用order by 字段作为唯一键。

Ø 去重不能跨分区。

Ø 只有合并分区才会进行去重。

Ø 认定重复的数据保留，版本字段值最大的。

Ø 如果版本字段相同则保留最后一笔。





### ***5.6***  ***S******ummingMergeTree***

 

对于不查询明细，只关心以维度进行汇总聚合结果的场景。如果只使用普通的MergeTree的话，无论是存储空间的开销，还是查询时临时聚合的开销都比较大。

Clickhouse 为了这种场景，提供了一种能够“预聚合”的引擎，SummingMergeTree.

```sql
create table t_order_smt(
    id UInt32,
    sku_id String,
    total_amount Decimal(16,2) ,
    create_time  Datetime 
 ) engine =SummingMergeTree(total_amount)
 partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id,sku_id );

   insert into  t_order_smt
values(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00')
(102,'sku_002',12000.00,'2020-06-01 13:00:00')
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```







## ***第六章  SQL操作***

   基本上来说传统关系型数据库（以MySQL为例）的SQL语句，基本支持但是也有不一样的地方。这里不会从头讲解SQL语法只介绍Clickhouse与标准SQL（MySQL）不一致的地方。

 

### ***6.1*** ***I******nsert*** 

 基本与标准SQL（MySQL）基本一致

包括标准 insert into  [table_name] values(…),(….) 

以及 从表到表的插入 

​     insert into  [table_name] select a,b,c from [table_name_2]

### ***6.2 Update*** ***和*** ***Delete*** 

ClickHouse提供了Delete 和Update的能力，这类操作被称为Mutation查询，它可以看做Alter 的一种。

虽然可以实现修改和删除，但是和一般的OLTP数据库不一样，***M******utation******语句是一种很“重”的操作，而且不支持事务***。

“重”的原因主要是每次修改或者删除都会导致放弃目标数据的原有分区，重建新分区。所以尽量做批量的变更，不要进行频繁小数据的操作。

删除操作

```sql
select * from t_order_smt where sku_id ='sku_002';

alter table t_order_smt delete where sku_id ='sku_002';
```



修改操作

```sql
select * from t_order_smt where id = 102

alter table t_order_smt 
update total_amount=toDecimal32(3000.00,2) 
where id =102;
```



由于操作比较“重”，所以 Mutation语句分两步执行，同步执行的部分其实只是进行新增数据新增分区和并把旧分区打上逻辑上的失效标记。知道触发分区合并的时候，才会删除旧数据释放磁盘空间。



```shell
clickhouse-client  --query    "select toHour(create_time) hr  ,count(*) from test1.order_wide where dt='2020-06-23'  group by hr" --format CSVWithNames > ~/rs1.csv


支持格式的地址
https://clickhouse.tech/docs/v19.14/en/interfaces/formats/#csvwithnames
```













