[toc]





## 1, timestamp介绍

> timestamp类型默认显示按照数据库的时区进行显示
>
> 类如数据库默认时区是UTC时区，那么timestamp记录的时间就是UTC时区的时间



### 1.1, 使用timestamp设置新增时间

```mysql
CREATE TABLE `user_ivanl001` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `the_num` int(11) DEFAULT NULL,
  `the_desc` varchar(255) DEFAULT NULL,
  `good` tinyint(1) DEFAULT NULL,
   -- 这里是新增时间，更新的时候是不会进行更新的
  `add_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
   -- 这里是更新时间，更新的时候是会自动进行更新的
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=62 DEFAULT CHARSET=utf8;
```



### 1.2, 使用timestamp设置更新时间

```mysql
CREATE TABLE `user_ivanl001` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `the_num` int(11) DEFAULT NULL,
  `the_desc` varchar(255) DEFAULT NULL,
  `good` tinyint(1) DEFAULT NULL,
   -- 这里是新增时间，更新的时候是不会进行更新的
  `add_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
   -- 这里是更新时间，更新的时候是会自动进行更新的
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=62 DEFAULT CHARSET=utf8;
```



## 2, 数据库更改时区

### 2.1, 查看数据当前时区

```mysql
# 查看时区
show variables like '%time_zone%';
# 查看时区当前时间
select now();
```



### 2.2, 更改时区相关

```mysql
1.2 修改时区
> set global time_zone = '+8:00'; ##修改mysql全局时区为北京时间，即我们所在的东8区
> set time_zone = '+8:00'; ##修改当前会话时区
> flush privileges; #立即生效


方法二：通过修改my.cnf配置文件来修改时区
# vim /etc/my.cnf ##在[mysqld]区域中加上
default-time_zone = '+8:00'
# /etc/init.d/mysqld restart ##重启mysql使新时区生效
```

