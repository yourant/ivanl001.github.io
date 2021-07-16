[TOC]



## 0, 今日任务

```mysql
20190924任务：
	1, cms报表部分统计了解
		任务状态：5%
	2， mysql采集系统调研测试，剩余一个更新es还在测试中
		任务状态：80%
20190925任务：
	1，mysql采集数据实时更新到es中
	
```



*　通用

```mysql
and (a02.email not like '%@tetx.com' and a02.email not like '%@i9i8.com' 
 and a02.email not like '%@qq.com' and a02.email not like '%@163.com' 
 and a02.email not like '%@126.com' and a02.email not like '%@jjshouse.com' 
 and a02.email not like '%@jenjenhouse.com' 
 and a02.email not like '%@azazie.com')
and a02.project_name = 'Azazie'
```



## 1, coming soon报表

* 根据sql分析得知这个计算的是
* 与goods_languages连接的目的就是去除商品名字

```mysql
select csr.goods_id goods_id, gl.goods_name goods_name, DATE_FORMAT(csr.add_time,'%Y-%m-%d') date, count(*) count
from coming_soon_remind_email csr
left join 
goods_languages gl 
on csr.goods_id=gl.goods_id
# english
where gl.languages_id=1
and csr.add_time>='2019-09-21 00:00:00'
and csr.add_time<'2019-09-21 00:00:00'
and (csr.user_email not like '%@tetx.com' and csr.user_email not like '%@i9i8.com' 
 and csr.user_email not like '%@qq.com' and csr.user_email not like '%@163.com' 
 and csr.user_email not like '%@126.com' and csr.user_email not like '%@jjshouse.com' 
 and csr.user_email not like '%@jenjenhouse.com' 
 and csr.user_email not like '%@azazie.com')
group by goods_id,date
order by date
```





## 2, mysql采集系统调研测试，剩余一个更新es还在测试中