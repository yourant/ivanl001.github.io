[TOC]

## 1, 认识数据库

### 01, DML

* 使用者角度， 80%， 

* 数据库处理数据等操作统称为数据操纵语言

### 02, DDL

* 建设者角度， 15%， 建表建库建视图
* 定义和管理数据库中所有对象的语言



### 03, DCL

* 管理员角度，DBA， 5%
* 授权， 事务发生时间和效果， 对数据库监控等



## 2, DML

### 01, insert操作

```mysql
MariaDB [test]> desc user;
+-------+----------------------+------+-----+---------+----------------+
| Field | Type                 | Null | Key | Default | Extra          |
+-------+----------------------+------+-----+---------+----------------+
| uid   | int(11)              | NO   | PRI | NULL    | auto_increment |
| name  | varchar(20)          | NO   |     |         |                |
| age   | smallint(5) unsigned | NO   |     | 0       |                |
+-------+----------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

# 插入全部字段的两种写法：
# 写法1:
insert into user values (2, 'ivanl002', 2);
# 写法2:
insert into user (uid, name, age) values (111, 'ivanl001', 100);

# 插入部分字段
insert into user (uid, name) values (1, 'ivanl001');


# 插入多行
insert into user (name, age) values ('ivanl005', 5), ('ivanl006', 6);
insert into user values (3, 'ivanl003', 3), (4, 'ivanl004', 4);

```



### 02, update操作

```mysql
# 单字段更新：
update user set age = 11 where uid = 1;
# 多字段更新：用逗号隔开，而不是and
update user set name = 'zhang', age = 11 where uid = 1;

# 所有的数据在原先的年龄上加10
update user set age = age+10;
```



### 03, delete操作

```mysql
# 因为mysql删除是原子性操作，只能完整的删除一行和多行，所以不需要写列名
delete from user where uid = 111;
```



### 04, select操作

#### 01, where基本

* where是表达式，判断where后表达式是真是假，如果是真就取出来，如果是假就不取

```mysql
select * from user;
select uid, name, age from user;
select * from user where age > 3;

# where后表达式恒为真，所以所有的数据都会取出来
select * from user where 1=1;

# 变量运算
select uid, name, age+1 from user order by uid;

# between是两边包
# 在 MySQL 中，BETWEEN 包含了 value1 和 value2 边界值

# -----floor用法
# 表中num如果在大于等于20，小于等于40，那么20几的改成20，30几的改成30
update mian set num = floor(num/10)*10 where num between 20 and 40;

# ----substring和concat用法
 select concat(substring(goods_name, 4), '---ivanl001') from goods where goods_name like '%诺基亚%';
```



#### null的判断

> mysql中判断null不能使用 = null或者 != null
>
> where后使用null=null其实取不出任何内容

```mysql 
# 这行结果是1，也就是true
select 1 = 1;

# 下面这两个都取不出东西，既不是true, 也不是false，结果是NULL，因为null和null无法比较
select null = null;
select null != null;
```

- 真正需要判断一个字段是否为空，需要使用如下方式：

```mysql 
select * from order where order_time is not null;
select * from order where order_time is null;
```





#### 02, group by和min, max, avg, count

* count(*)和count(1)效率一样，没有哪个比哪个更优化或更好
* group by 如果取的结果字段中有没有聚合的字段，那么会取组内第一个哦

```mysql
# 分组的时候mysql内部会对要分组字段先进行排序，然后再计算每个分组字段的聚合结果，比如说平均值等
select cat_id, avg(shop_price) from goods group by cat_id;
select cat_id, max(shop_price) from goods group by cat_id;
```



#### 03, having

* mysql数据存储在磁盘上
* 先读取到磁盘文件到内存，起别名就是内存中的操作了

* where是针对磁盘上的数据文件起作用，所以对于别名是识别不了的
* having是可以作用于内存中的数据，可以识别别名
* 所以having是对结果集进行筛选的，也就是内存中的数据进行筛选
* 所以having必须要写在where的后面，因为必须要先从磁盘读取到内存，然后再操作内存中的数据

```mysql
select goods_id, goods_name, market_price-shop_price from goods;
# where
select goods_id, goods_name, market_price-shop_price from goods where (market_price-shop_price) > 200;
# 这样用别名是报错的，说where中没有save字段
select goods_id, goods_name, (market_price-shop_price) as save from goods where save > 200;

# having可以作用于内存中的，也就是别名
select goods_id, goods_name, (market_price-shop_price) as save from goods having save > 200;
```



#### 综合案例

> 从如下表中选出两门及两门不及格学生的平均成绩

```mysql
MariaDB [test]> select * from stu;
+------------+------------+-------+
| name       | subject    | score |
+------------+------------+-------+
| 张三       | 数学       |    90 |
| 张三       | 语文       |    50 |
| 张三       | 地理       |    40 |
| 李四       | 语文       |    55 |
| 李四       | 政治       |    45 |
| 王五       | 政治       |    30 |
| 5          | liqianqian |    10 |
| liqianqian | 英语       |    10 |
+------------+------------+-------+
8 rows in set (0.00 sec)
```



```mysql 
# 注意：sum这里， 不能直接用count(score<60), 因为count是计数，不管里面是对是错，都是会计的
# if(score<60, 1, 0)可以直接改为 score<60
select name, sum(if(score<60, 1, 0)) as count, avg(score) from stu group by name having count>=2;
select name, sum(score<60) as count, avg(score) from stu group by name having count>=2;

# 上面的代码也可以改成下面这个
# 子查询
select a01.name, a01.counter, a01.avg
from 
(select name, sum(if(score<60, 1, 0)) as counter, avg(score) as avg from stu group by name) as a01
where counter >= 2;
```



#### 04, order by

* asc-升序，默认
* desc-降序

```mysql

```



#### 05, limit

* limit x, y  从x+1条开始取， 取y条数据
* limit可以用于分页

```mysql
# 从第二条开始取，取出5条数据出来
select * from goods order by shop_price desc limit 1, 5;
```



#### 总结和子查询

* 上面五种子句有严格的顺序：
* where, group by, having, order by, limit

```mysql
# 找出最新商品，也就是good_id最大的商品
# 第一种方式：order by 
select * from goods order by goods_id desc limit 1;


# 但是如果没有索引的话，排序是很消耗性能的
# 第二种方式:where子查询
select * from goods where goods_id = (select max(goods_id) from goods)
```



```mysql
MariaDB [test]> select goods_id, goods_name, cat_id, shop_price, market_price from goods;
+----------+----------------------------------------+--------+------------+--------------+
| goods_id | goods_name                             | cat_id | shop_price | market_price |
+----------+----------------------------------------+--------+------------+--------------+
|       32 | 诺基亚n85                              |      3 |    3010.00 |      3612.00 |
+----------+----------------------------------------+--------+------------+--------------+
31 rows in set (0.00 sec)


# 找出每个类别最新的商品， good_id大的就是最新的商品
# 方式一：where子查询
select goods_id, goods_name, cat_id, market_price from goods where goods_id in (select max(goods_id) as goods_id from goods group by cat_id) order by cat_id;

# 方式二：连接查询
select a02.goods_id, a02.goods_name, a02.cat_id, a02.market_price
from
(select max(goods_id) as goods_id, cat_id from goods group by cat_id) as a01
join 
(select goods_id, goods_name, cat_id, market_price from goods) as a02
on a01.goods_id = a02.goods_id and a01.cat_id = a02.cat_id
order by a02.cat_id;
```



##### where子查询

```mysql
# 选择最新的商品
select cat_id, goods_id, goods_name from goods where goods_id = (select max(goods_id) from goods);
```



##### from子查询

```mysql
# group by不是取第一个吗，这里好像不对。group取出来的非聚合字段应该是不加任何修饰下第一次出现，排过序无效
# 这里不对，可能是mysql不同版本的原因
# 找出每个类别最新的商品， good_id大的就是最新的商品，所以这里不能选出每个类别下最新的商品
select cat_id, goods_id, goods_name from (select * from goods where 1 order by cat_id asc, goods_id desc) as tmp group by cat_id;
```



##### exists型子查询

```mysql
# 查询出不存在商品的类别
select * from category where not exists (select * from goods where goods.cat_Id = category.cat_id);
```



#### 06, join

##### 01, inner join

> join其实默认就是inner join

```mysql
select b.bname, g.gname from boy as b join girl as g on b.hid = g.hid;
```



##### 02, left join

```mysql
select b.bname, g.gname from boy as b left join girl as g on b.hid = g.hid;
```



##### 03, right join

```mysql
select b.bname, g.gname from boy as b right join girl as g on b.hid = g.hid;
```



##### 04, outer join(mysql中没有这个)

> mysql中没有outer join, hive中左边没有左边为null，右边没有右边为null

```mysql
select b.bname, g.gname from boy as b outer join girl as g on b.hid = g.hid;
```



#### 07, union

> 使用union时候，完全相同的行会被合并
>
> 合并是比较耗时的操作
>
> 一般完全相同的数据也不希望被合并，那么就不能使用union合并，而是使用union all，这样即使是完全相同的数据也不会被合并掉
>
> union的子句中，不用写order by，因为不会起作用。最需要在最后 的结果集中进行排序即可



## 3, DDL

### 01, 整型

作为SQL标准的扩展，MySQL也支持整数类型TINYINT、MEDIUMINT和BIGINT。下面的表显示了需要的每个整数类型的存储和范围。

| **类型**  | **字节** |          | **最小值**              | **最大值**              |
| --------- | -------- | -------- | ----------------------- | ----------------------- |
|           |          |          | **(带符号的/无符号的)** | **(带符号的/无符号的)** |
| TINYINT   | 1        | 默认     | -128                    | 127                     |
|           |          | unsigned | 0                       | 255                     |
| SMALLINT  | 2        | 默认     | -32768                  | 32767                   |
|           |          | unsigned | 0                       | 65535(6万)              |
| MEDIUMINT | 3        | 默认     | -8388608                | 8388607                 |
|           |          | unsigned | 0                       | 16777215(1600万)        |
| INT       | 4        | 默认     | -2147483648             | 2147483647              |
|           |          | unsigned | 0                       | 4294967295(42亿)        |
| BIGINT    | 8        | 默认     | -9223372036854775808    | 9223372036854775807     |
|           |          | unsigned | 0                       | 18446744073709551615    |

* 如果存储的数值超过规定的数值， 可以存储进去，但是结果可能会溢出造成数据错误
* zerofill: 长度一定，默认填充为0， 填充的宽度也就是int(10)括号中的长度
* 注意：整型括号内的数字并不是限制数字的大小，而是在zerofill的时候指定一下填充的长度而已

```mysql
# 无符号，最大127，超过的话会溢出造成数据有误
create table t1 (tinyNum tinyint);
# 无符号
alter table t1 add untinyNum tinyint unsigned;

# 自动填充0,填充的长度是5
# 注意：设置zerofill后，该类型自动变成unsigned
alter table t1 add num03 tinyint(5) zerofill;
# 如果插入2的话得到结果：00002
```



### 02, 浮点型

> float, double有精度损失，所以一般不用
>
> decimal， 是定点型，没有精度损失
>
> decimal(5, 2)表示一共5位，小数占2位



```mysql
create table t2 ( f float(9,2), d decimal(9,2));
insert into t2 (1234567.35, 1234567.35);
select * from t2;

# 可以看到float或者double，是有精度损失的
# decimal则没有
+------------+------------+
| f          | d          |
+------------+------------+
| 1234567.38 | 1234567.35 |
+------------+------------+
```



### 03, 字符型

> char是定长
>
> varchar是变长

下面的表显示了将各种字符串值保存到CHAR(4)和VARCHAR(4)列后的结果，说明了CHAR和VARCHAR之间的差别：

*　varchar是变长，所以需要占用额外的一个字节存储长度
*　如果字符长度固定，用char比较好
*　如果字符不固定，但是最大不超过10，用char合适
*　如果字符长度不固定，但是一般也比较长，比如说50， 100长度，等，用varchar
*　相对而言，char定长，速度会更快一些 

| **值**     | CHAR(4) | **存储需求** | VARCHAR(4) | **存储需求** |
| ---------- | ------- | ------------ | ---------- | ------------ |
| ''         | '    '  | 4个字节      | ''         | 1个字节      |
| 'ab'       | 'ab  '  | 4个字节      | 'ab '      | 3个字节      |
| 'abcd'     | 'abcd'  | 4个字节      | 'abcd'     | 5个字节      |
| 'abcdefgh' | 'abcd'  | 4个字节      | 'abcd'     | 5个字节      |

请注意上表中最后一行的值只适用*不使用严格模式*时；如果MySQL运行在严格模式，超过列长度不的值*不保存*，并且会出现错误。

char(10), varchar(10)中的10都是限制的字符长度，如果是utf-8,就是允许10个utf8字符

char的长度最大是255，varchar目前最大是65535



TEXT[(*M*)]

最大长度为65,535(216–1)字符的TEXT列。

可以给出可选长度*M*。则MySQL将列创建为最小的但足以容纳*M*字符长的值的TEXT类型。



MEDIUMTEXT

最大长度为16,777,215(224–1)字符的TEXT列。



LONGTEXT

最大长度为4,294,967,295或4GB(232–1)字符的TEXT列。LONGTEXT列的最大*有效*(允许的)长度取决于客户端/服务器协议中配置最大包大小和可用的内存。



ENUM('*value1*','*value2*',...)

枚举类型。只能有一个值的字符串，从值列'*value1*'，'*value2*'，...，NULL中或特殊  ''错误值中选出。ENUM列最多可以有65,535个截然不同的值。ENUM值在内部用整数表示。



SET('*value1*','*value2*',...)

一个设置。字符串对象可以有零个或多个值，每个值必须来自列值'*value1*'，'*value2*'，...SET列最多可以有64个成员。SET值在内部用整数表示。  



```mysql

```



### 04, 日期时间

*　timestamp效率并不高，所以一般也不建议使用
*　如果需要使用非常精确的时间，可以使用int unsigned
*　不需要特别精确，可以使用datetime 



Year     1990

Date    1990-02-07

Time    18:08:08

datetime   1990-02-07 18:08:08

timestamp   1990-02-07 18:08:08   有默认值的datetime



```mysql
CREATE TABLE t3 (id int, ts TIMESTAMP);

insert into t3 (id) values (1);
select * from t3;

# 可以看到timestamp默认就是会把当前时间设置上
+------+---------------------+
| id   | ts                  |
+------+---------------------+
|    1 | 2019-09-23 17:23:01 |
+------+---------------------+
```



### 05, 列的默认值

> 1, NULL查询不方便
>
> 2, NULL的索引效率不高
>
> 所以在使用中，一般都避免使用NULL

```mysql
# 设置默认值
create table t4 (id int not null default 0, name char(10) not null default '');

insert into t4 (id) values (1);
insert into t4 (id, name) values (2, 'ivanl002');


MariaDB [test]> select * from t4 where name is not null;
+----+----------+
| id | name     |
+----+----------+
|  1 |          |
|  2 | ivanl002 |
+----+----------+
2 rows in set (0.00 sec)

```



### 06, 创表和主键

```mysql
# 下面两句等效
create table t5 (id int primary key, name char(2));
create table t6 (id int, name char(2), primary key(id));

# 给id加上自增属性
# auto_increment：一张表只能有一列设置成auto_increment， 此列必须加索引
create table t7 (id int primary key auto_increment, name char(2));
```



### 07, 列的增删改

```mysql
# 增加字段
alter table t8 add height tinyint unsigned not null default 0;

# 删除字段
alter table t8 drop column height;

# 插入字段
alter table t8 add height tinyint unsigned not null after id;
 
# 更改字段
alter table t8 change height shengao smallint;
alter table t8 modify shengao tinyint; 
```



### 08, 视图

```mysql
create view   as select * from a;
```



### 09, 表视图管理语句

```mysql
show create table a;

desc a;

# 表重改名
rename table a to a01;

# 清空表
truncate 

# 清空表
truncate a01;

# 删除表
drop table a01;

# 查看所有表的状态
show table status \G;

# 查看某张表的状态
show table status where name = 'a01' \G;
```



### 10, 表的数据存储

>  InnoDB: 默认的数据库引擎, 每张表只有一个文件.frm属性文件，然后所有表的数据文件都存储在一个文件中， 存在/var/lib/mysql根目录下ibdata1
>
> myisam: 之前的数据库引擎， 每张表都会有一个.frm列属性文件，myd数据文件和myi索引文件(有索引的话)， 存在每个数据库文件夹下

```mysql
# frm是表的列的属性
# MYD是表的数据
# MYI是表的索引
-rw-rw---- 1 mysql mysql 8898 Sep 23 08:49 goods.frm
-rw-rw---- 1 mysql mysql 1696 Sep 23 08:49 goods.MYD
-rw-rw---- 1 mysql mysql 2048 Sep 23 09:36 goods.MYI
```



**存储引擎的选择**

不同的存储引擎都有各自的特点，以适应不同的需求，如下表所示：

| **功 能**    | **MYISAM** | **InnoDB** | BDB    | **Memory** | **Archive** |
| ------------ | ---------- | ---------- | ------ | ---------- | ----------- |
| 批量插入速度 | 快         | 慢         | 高     | 高         | 非常高      |
| 存储限制     | 256TB      | 64TB       | 无     | RAM        | 无          |
|              | 小         | 大         | 小     | 中等       | 小          |
| 支持事物     | No         | 支持       | 支持   | No         | No          |
| 支持全文索引 | 支持       | No         | 不支持 | No         | No          |
| 锁机制       | 表锁       | 行锁       | 页锁   | 表锁       | 行锁        |
| B树索引      |            | 支持       |        |            |             |
| hash索引     | No         | 支持       |        | Yes        | No          |
| 集群索引     |            | 支持       |        |            |             |
| 数索引       | Yes        | Yes        |        | Yes        | No          |
| 数据缓存     | No         | 支持       |        | N/A        | No          |
| 索引缓存     |            | 支持       |        |            |             |
| 支持外键     | No         | Yes        |        | No         | No          |

如果要提供提交、回滚、崩溃恢复能力的事物安全（ACID兼容）能力，并要求实现并发控制，InnoDB是一个好的选择

如果数据表主要用来插入和查询记录，则MyISAM引擎能提供较高的处理效率

如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存中的Memory引擎，MySQL中使用该引擎作为临时表，存放查询的中间结果

如果只有INSERT和SELECT操作，可以选择Archive，Archive支持高并发的插入操作，但是本身不是事务安全的。Archive非常适合存储归档数据，如记录日志信息可以使用Archive

使用哪一种引擎需要灵活选择，**一个数据库中多个表可以使用不同引擎以满足各种性能和实际需求**，使用合适的存储引擎，将会提高整个数据库的性能





### 11, 字符集

```mysql
# 这个是客户端的字符集
set character_set_client=utf8
# 这个是数据库中的字符集
set character_set_results=utf8
# 这个是连接器的字符集
set character_set_connection=utf8
```



### 12, 索引

> 极大提高查询速度，但是会降低插入速度

#### 01, key 普通索引

*　加快查询速度

#### 02, unique key  唯一索引

* 加快查询速度， 并限制索引需要唯一
* 与"普通索引"类似，不同的就是：索引列的值必须唯一，但允许有空值。

#### 03, primary key  主键索引 

* 一张表中只能有一个
* 不允许有空值

#### 04, fulltext  全文索引

* 在中文下几乎无效



```mysql

create table t9 (name char(10), email char(20), key name(name), unique key email(email));

create table t10 (id int, name char(10), email char(20), 
                 key name(name), 
                 unique key email(email),
                 primary key(id));
                 
# 比如对邮箱进行所索引的时候，一般后面的索引度就不高
# 所以可以只对@前面的信息进行索引
# 下面就是只对邮箱的前10个字符进行索引
create table t11 (name char(10), email char(20), key name(name), unique key email(email(10)));
```



#### 05, 多列索引

* 多列合并起来进行索引

```mysql
create table t12 (id int, firstName char(10), secondName char(10), primary key (id), key name(firstName, secondName))

# 插入一条数据方便测试
insert into t12 (firstName, secondName) values ('ivanl', 'bool');

# 这里可以看到where firstName = 'ivanl' and secondName = 'bool'的时候，索引有效
MariaDB [test]> explain select * from t12 where firstName = 'ivanl' and secondName = 'bool' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t12
         type: ref
possible_keys: name
          key: name
      key_len: 62
          ref: const,const
         rows: 1
        Extra: Using where; Using index
1 row in set (0.00 sec)

ERROR: No query specified

# where firstName = 'ivanl'的时候索引有效
MariaDB [test]> explain select * from t12 where firstName = 'ivanl' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t12
         type: ref
possible_keys: name
          key: name
      key_len: 31
          ref: const
         rows: 1
        Extra: Using where; Using index
1 row in set (0.00 sec)

ERROR: No query specified

# 但是where secondName = 'bool'的时候索引就无效了哦
MariaDB [test]> explain select * from t12 where secondName = 'bool' \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t12
         type: index
possible_keys: NULL
          key: name
      key_len: 62
          ref: NULL
         rows: 1
        Extra: Using where; Using index
1 row in set (0.00 sec)

ERROR: No query specified

```



#### 06,  冗余索引

* 在某个列上可能存在多个索引

```mysql
# secondName这一列上既有name索引，又有second索引
create table t13 (id int, firstName char(10), secondName char(10), primary key (id), key name(firstName, secondName), key second(secondName))

# 查看索引
show index from t13 \G;
```



#### 07, 索引操作

```mysql
# 创建索引见上面的内容

# 查看索引
show index from t13 \G;
show create table t13;

# 删除索引
alter table t13 drop index second;
drop index name on t13;

# 添加索引
alter table t13 add index name(firstName, secondName);
alter table t13 add unique index second(secondName);

# 添加主键索引，因为主键索引全表只有一个，所以不需要起名字
alter table t13 add primary key(id);

# 删除主键索引
alter table t13 drop primary key;
```









```mysql
# 只复制表结构
create table stu like result;

# 复制表结构和数据
create table stu select * from result;
```

