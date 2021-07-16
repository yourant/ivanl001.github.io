[toc]



# 1, explain具体结果解析

![image-20200201094811192](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201094811192.png)



## 1.0, 创表测试表语句 

```mysql
create table t1(id INT(10) AUTO_INCREMENT, content varchar(100) NULL, PRIMARY KEY(id));
create table t2(id INT(10) AUTO_INCREMENT, content varchar(100) NULL, PRIMARY KEY(id));
create table t3(id INT(10) AUTO_INCREMENT, content varchar(100) NULL, PRIMARY KEY(id));
create table t4(id INT(10) AUTO_INCREMENT, content varchar(100) NULL, PRIMARY KEY(id));

insert into t1(content) values (concat('t1', floor(1+rand()*1000)));
insert into t2(content) values (concat('t2', floor(1+rand()*1000)));
insert into t3(content) values (concat('t3', floor(1+rand()*1000)));
insert into t4(content) values (concat('t4', floor(1+rand()*1000)));
```



## 1.1, id

* id主要是描述所需要查询的表的查询顺序
* 这个不是很重要



> id相同的时候，执行顺序由上到下
>
> id不同，id的值越大，则会优先查询。



> id有多少个不同的数字就会有多少趟查询，趟数越少越好，效率越高



* id相同的时候

  ![image-20200201111822141](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201111822141.png)

* id不同（高版本的sql会自动优化，在我测试的时候下面这个sql的所有id都是相同的）

  ![image-20200201111733211](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201111733211.png)



## 1.2, select_type

| Select_type的值      | 具体的意义                                                   |
| -------------------- | ------------------------------------------------------------ |
| SIMPLE               | 简单的select查询，没有用到union或者自查询                    |
| PRIMARY              | 查询中包括任何复杂的子部分，则最外部的查询则标记为PRIMARY查询 |
| UNION                | union查询                                                    |
| DEPENDENT UNION      |                                                              |
| UNION RESULT         | union的结果                                                  |
| SUBQUERY             | 在SELECT或WHERE列表中包含了子查询                            |
| DEPENDENT SUBQUERY   |                                                              |
| DERIVED              | 衍生表查询, 在from列表中包含的子查询被标记为DERIVED          |
| UNCACHEABLE SUBQUERY | 无法进行缓存的字查询(例如条件是随机值)                       |
| UNCACHEABLE UNION    | 无法进行缓存的union查询                                      |

![image-20200201110958030](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201110958030.png)



![image-20200201111144464](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201111144464.png)



![image-20200201111228088](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201111228088.png)



![image-20200201111344802](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201111344802.png)



![image-20200201111523873](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201111523873.png)



![image-20200201111617793](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201111617793.png)



## 1.3, table

无需多讲，就是查询的是哪张表



## 1.4, type

> type从最好到最次依次是：
>
> system(平时不会出现) > const(匹配一行数据) > eq_ref > ref > range > index > ALL



![image-20200201112014275](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201112014275.png)





## 1.5, possible_keys

显示可能应用在这张表中的索引，一个或多个。

查询涉及到的字段上若存在索引，则该索引将被列出，**但不一定被查询实际使用**



## 1.6, key

实际使用的索引。如果为NULL，则没有使用索引

查询中若使用了覆盖索引，则该索引和查询的select字段重叠



## 1.7, key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。

key_len字段能够帮你检查是否充分的利用上了索引



如何计算

1 、先看索引上字段的类型+长度比如 int=4 ;  varchar(20 ) =20 ; char(20 ) =20  

2 、如果是varchar或者char这种字符串字段，视字符集要乘不同的值，比如utf-8 要乘 3,GBK要乘2，

3 、varchar这种动态字符串要加2个字节

4、 允许为空的字段要加1个字节 



![image-20200201195238021](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201195238021.png)



## 1.8, ref

显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值



## 1.9, rows

rows列显示MySQL认为它执行查询时必须检查的行数。



## 1.10, Extra

包含不适合在其他列中显示但十分重要的额外信息

![image-20200201202808441](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201202808441.png)



覆盖索引：

一般查询先是查询索引，然后通过索引进行回表查询。但是在某些情况下满足：查到索引后，可以直接获取所需要的值(这列数据做过索引，所以可以直接取出)，这样的方式就是覆盖索引， 也就是查询列要么是索引列，要么是主键。覆盖索引在extra中总是会有USING INDEX字段

https://blog.csdn.net/qq_15037231/article/details/87891683





# 2, 索引优化相关



## 2.1, 索引失效

在索引列上的renew操作，包括类型转换，函数，计算等等，都会导致索引失效

如果创建复合索引，然后使用索引的时候，中间索引使用到了范围，那么后续的索引都会失效

msyql使用不等于，会使索引失效

is not null无法使用索引

like的%放在最前会使索引失效

字符串不加单引号(这个其实是索引列上进行了类型转换)，会使索引失效

![image-20200201211351202](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201211351202.png)



## 2.2, 索引优化建议



![image-20200201211533508](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201211533508.png)



## 2.3, 关联时候索引优化

小表作为驱动表，大表作为被驱动表，这样，小表只要排出一小部分数据，大表就能排出掉一大部分数据





![image-20200201211628930](/Users/ivanl001/Documents/buer/learning_notebook/01-im_bool_notebook/bool_06_big_data/asserts/image-20200201211628930.png)



## 2.4, 子查询优化

尽量不要使用not in 或者 not exists

用left outer join on xxx is null 替代



## 2.4, 排序分组优化

**结论** ：  **当范围条件和****group by** **或者** **order by** **的字段出现二选一时** **，** **优先观察条件字段的过滤数量** **，如果过滤的数据足够多，而需要排序的数据并不多时，优先把索引放在范围字段上。反之，亦然。** 



## 2.5, 慢查询

之前记录过，这里就不记录了



# 3, 主从复制

主从复制其实就是同过binlog进行复制的。之前也总结过 ，这里不再赘述