[TOC]

## 0, 今日报表

```mysql
20190918任务：
	1，flash sale剩余的几个指标完成分析
		任务状态：完成
	2，实时数据报表的大数据方案定案
		任务状态：未开始(因为临时来了下面1，2，3的任务，所以只能后续安排这个)
	3，生产数据库的进一步熟悉
		任务状态：进行中
	4, 已付款且包含裙子的订单中商品数量情况
		任务状态：已完成
	5, Live  Date原先方案解析
		任务状态：已完成
	6, flash sale的销量
		任务状态：已完成
20190919任务：
	1，实时数据报表的大数据方案定案
	2，大数据系统导入导出数据平台模拟
	3，如果时间还有，需要进行模拟数据生成
```



## 1, 先解决Event Date需求

* 数据存在问题，暂时不能进行区分是否有Event Date
* 需要后台人员后续更新代码后才能识别

```mysql
# 154375
select count(1) from order_info where and order_status = 2 order_time >= '2018-09-01 00:00:00' and order_time <  '2019-09-01 00:00:00' 

# 3634613
select count(1) from order_goods


# 有过订单的用户数
# users中的country很多是不正确的，所以这里使用user_address，如果用户不存在user_address表中，那么就代表不在我们的统计范围之内
select a02.country, count(distinct a01.user_id)
from 
(select distinct user_id from order_info where pay_status = 2 and pay_time >= '2018-09-01 00:00:00' and pay_time < '2019-09-01 00:00:00'
  and project_name = 'Azazie'
  and (email not like '%@tetx.com' and email not like '%@i9i8.com' 
  	and email not like '%@qq.com' and email not like '%@163.com' 
  	and email not like '%@126.com' and email not like '%@jjshouse.com' 
  	and email not like '%@jenjenhouse.com' and email not like '%@azazie.com')) as a01
join
(select user_id, country from user_address where address_type = 'SHIPPING' and is_default = 1 and country in (3859,3844)) as a02
on a01.user_id = a02.user_id
group by country;



# 下单用户中选择了具体Event Date的用户
select b02.country, count(distinct b01.user_id)
from 
(select distinct a01.user_id
from
(select user_id, order_id from order_info where pay_status = 2 and order_time >= '2018-09-01 00:00:00' and pay_time < '2019-09-01 00:00:00'
  and project_name = 'Azazie'
  and (email not like '%@tetx.com' and email not like '%@i9i8.com' 
  	and email not like '%@qq.com' and email not like '%@163.com' 
  	and email not like '%@126.com' and email not like '%@jjshouse.com' 
  	and email not like '%@jenjenhouse.com' and email not like '%@azazie.com')) as a01
left join 
(select order_id from order_extension where ext_name = 'event_date' and ext_value != '0000-00-00') as a02
on a01.order_id = a02.order_id
where a02.order_id is not NULL) as b01

join

(select user_id, country from user_address where address_type = 'SHIPPING' and is_default = 1 and country in (3859,3844)) as b02
on b01.user_id = b02.user_id
group by b02.country;



# 上面两个的join起来
#  数据本身存在问题，所以就不需要再做接下来的数据了
select c01.country, c01.userCount, c02.userCountWithEventDate
from

(select a02.country, count(distinct a01.user_id) as userCount
from 
(select distinct user_id from order_info where pay_status = 2 and pay_time >= '2018-09-01 00:00:00' and pay_time < '2019-09-01 00:00:00'
  and project_name = 'Azazie'
  and (email not like '%@tetx.com' and email not like '%@i9i8.com' 
  	and email not like '%@qq.com' and email not like '%@163.com' 
  	and email not like '%@126.com' and email not like '%@jjshouse.com' 
  	and email not like '%@jenjenhouse.com' and email not like '%@azazie.com')) as a01
join
(select user_id, country from user_address where address_type = 'SHIPPING' and is_default = 1 and country in (3859,3844)) as a02
on a01.user_id = a02.user_id
group by country) c01

left join 

(select b02.country, count(distinct b01.user_id) as userCountWithEventDate
from 
(select distinct a01.user_id
from
(select user_id, order_id from order_info where pay_status = 2 and pay_time >= '2018-09-01 00:00:00' and pay_time < '2019-09-01 00:00:00'
  and project_name = 'Azazie'
  and (email not like '%@tetx.com' and email not like '%@i9i8.com' 
  	and email not like '%@qq.com' and email not like '%@163.com' 
  	and email not like '%@126.com' and email not like '%@jjshouse.com' 
  	and email not like '%@jenjenhouse.com' and email not like '%@azazie.com')) as a01
left join 
(select order_id from order_extension where ext_name = 'event_date' and ext_value != '0000-00-00') as a02
on a01.order_id = a02.order_id
where a02.order_id is not NULL) as b01

join

(select user_id, country from user_address where address_type = 'SHIPPING' and is_default = 1 and country in (3859,3844)) as b02
on b01.user_id = b02.user_id
group by b02.country) as c02
on 
c01.country = c02.country
```





| **country** | **dress_count_1** | **dress_count_2** | **dress_count_3** |
| ----------- | ----------------- | ----------------- | ----------------- |
| CA          | **296**           | **152**           | **296**           |
| US          |                   |                   |                   |

* Event date销量相关需求



### 已付款且包含裙子的订单中商品数量情况

```mysql
select if(c01.country=3844, 'CA', 'US') as country, 
sum(dress_count_1) as dress_count_1, 
sum(dress_count_2) as dress_count_2, 
sum(dress_count_3) as dress_count_3
from 

(select b01.user_id, b01.order_id, b01.dress_count, b02.country, 
if(b01.dress_count = 1, 1, 0) as dress_count_1,
if(b01.dress_count = 2, 1, 0) as dress_count_2,
if(b01.dress_count >= 3, 1, 0) as dress_count_3
from

(select a02.user_id, a02.order_id, sum(a01.goods_number) as dress_count from 
order_goods as a01
left join 
order_info as a02
on a01.order_id = a02.order_id
left join 
goods as a03
on a01.goods_id = a03.goods_id
where a02.pay_time >= '2018-09-01 00:00:00' and a02.pay_time < '2018-10-01 00:00:00' 
and a02.pay_status=2 and a02.project_name = 'Azazie'
and (a02.email not like '%@tetx.com' and a02.email not like '%@i9i8.com' 
	and a02.email not like '%@qq.com' and a02.email not like '%@163.com' 
	and a02.email not like '%@126.com' and a02.email not like '%@jjshouse.com' 
	and a02.email not like '%@jenjenhouse.com' 
	and a02.email not like '%@azazie.com')
and a03.cat_id in (2,7,8,9)
group by a02.user_id, a02.order_id) as b01
join
(select user_id, country from user_address where address_type = 'SHIPPING' and is_default = 1 and country in (3859,3844)) as b02

on b01.user_id = b02.user_id) as c01

group by c01.country
```



## 2，Live  Date原先方案解析

### 01,  前端

*　代码定位：app/tpl/admin/orderNumber/realtimeStat.htm

### 02, 后端逻辑

* 代码定位：app/tpl/admin/orderNumber/realtimeStat.htm

* 具体方法：private function _realtimeStat($date, $exclude_cancelled_orders = false)

*　az项目代码具体：839行
*　解析在代码中:
  *　简单说明：
  *　其实就是从数据库中读取当天的数据， 按照小时进行分组，然后每个小时的数据存储起来
  *　

```mysql
select
  # 时间，当天的第几个小时
  hour(pay_time) as compute_date,
  # 这个是每小时总订单数
  count(*) as average1day,
  # 这个是每小时总金额
  ifnull(sum(goods_amount), 0) as amount1day,
  # 这个代表付钱金额不是0的订单个数
  sum(if(order_amount > 0, 1, 0)) as real_order_number_1day,
  # 这个代表付款呢金额是0的，也就是免单的订单个数
  sum(if(order_amount = 0, 1, 0)) as free_order_number_1day,

  sum(ifnull((
   select sum(og.goods_number)
      from order_goods og
          left join goods g on g.goods_id = og.goods_id
      where og.order_id = a00.order_id and g.cat_id in (2, 7, 8, 9, 45, 158)
  ),0)) as dress1day,

    # Dresses
  sum(ifnull((
   select sum(og.goods_number)
      from order_goods og
      INNER JOIN goods ON goods.goods_id = og.goods_id
      where og.order_id = a00.order_id and (og.goods_id = 1000529 OR goods.cat_id = 206) and og.styles not like '%"isClearance":"1"%'
  ),0)) as sample1day,

  sum(ifnull((
   select sum(og.goods_number)
      from order_goods og 
      inner join goods  on goods.goods_id = SUBSTRING_INDEX(substring(og.styles, LOCATE('"rent_goods_style_id":"', og.styles) + 23),'"',1)
      where goods.cat_id in (7,45, 139, 158) and og.order_id = a00.order_id and og.goods_id = 1001199 and og.styles not like '%"isClearance":"1"%'
  ),0)) as sampleBridesmaid1day,

  sum(ifnull((
   select sum(og.goods_number)
      from order_goods og 
      inner join goods  on goods.goods_id = SUBSTRING_INDEX(substring(og.styles, LOCATE('"rent_goods_style_id":"', og.styles) + 23),'"',1)
      where goods.cat_id in (8) and og.order_id = a00.order_id and og.goods_id = 1001199 and og.styles not like '%"isClearance":"1"%'
  ),0)) as sampleMotherBridal1day,

  sum(ifnull((
   select sum(og.goods_number)
      from order_goods og
      inner join goods on goods.goods_id = SUBSTRING_INDEX(substring(og.styles, LOCATE('"rent_goods_style_id":"', og.styles) + 23),'"',1)
      where goods.cat_id = 2 and og.order_id = a00.order_id and og.goods_id = 1001199  and og.styles not like '%"isClearance":"1"%'
  ),0)) as sampleBridal1day,

  # swatch1day
  sum(ifnull((
   select sum(og.goods_number)
      from order_goods og
      where og.order_id = a00.order_id and og.goods_id in (1000291, 1000354, 1001248, 1001284)
  ),0)) as swatch1day,

  sum(ifnull((
    select sum(og.goods_number)
      from order_goods og
      INNER JOIN goods ON goods.goods_id = og.goods_id
      where og.order_id = a00.order_id and (og.goods_id = 1000529 OR goods.cat_id = 206) and og.styles like '%"isClearance":"1"%'
  ),0)) as clearance1day,


 # dress_bridal_real_1day
  sum(ifnull((
          select sum(og.goods_number)
          from order_goods og 
          inner join goods on og.goods_id = goods.goods_id
          where a00.order_id = og.order_id and goods.cat_id = 2
       ),0)) as dress_bridal_real_1day,


   sum(ifnull((
          select sum(og.goods_number)
          from order_goods og 
          inner join goods on og.goods_id = goods.goods_id
          where a00.order_id = og.order_id and goods.cat_id = 8
       ),0)) as dress_mob_real_1day,


  sum(ifnull((
          select sum(og.goods_number)
          from order_goods og 
          inner join goods on og.goods_id = goods.goods_id
          where a00.order_id = og.order_id and goods.cat_id in (7, 9, 45, 158)
       ),0)) as dress_bridesmaid_real_1day,

    sum(ifnull((
      select sum(og.goods_number)
      from order_goods og 
      inner join goods on og.goods_id = goods.goods_id
      where a00.order_id = og.order_id and goods.cat_id in (6, 13, 15, 138, 139)
   ),0)) as accessory_real_1day,


   sum(ifnull((
      select sum(og.goods_number)
      from order_goods og 
      inner join goods on og.goods_id = goods.goods_id 
      where a00.order_id = og.order_id and goods.cat_id in ( 13, 15, 139)
   ), 0)) as separate_wrap_sash_real_1day,


   sum(ifnull((
      select sum(og.goods_number)
      from order_goods og
      inner join goods on goods.goods_id = og.goods_id 
      where a00.order_id = og.order_id and goods.cat_id = 6
   ), 0)) as veil_real_1day, 


   sum(ifnull((
      select sum(og.goods_number)
      from order_goods og
      inner join goods on og.goods_id = goods.goods_id 
      where a00.order_id = og.order_id and goods.cat_id = 138
   ), 0)) as groomsmen_real_1day,


   sum(ifnull((
      select sum(og.goods_number)
      from order_goods og
      inner join goods on og.goods_id = goods.goods_id 
      where og.order_id = a00.order_id and goods.cat_id = 206 and og.styles->'$.select.isClearance' = '1'
   ), 0))as clearance_real_1day


from order_info as a00
where pay_time >= '2019-09-16 00:00:00' and pay_time < '2019-09-17 00:00:00' 
and pay_status=2 and project_name = 'Azazie'
and (email not like '%@tetx.com' and email not like '%@i9i8.com' 
	and email not like '%@qq.com' and email not like '%@163.com' 
	and email not like '%@126.com' and email not like '%@jjshouse.com' 
	and email not like '%@jenjenhouse.com' 
	and email not like '%@azazie.com')
and (email not in ('pxing@i9i8.com','xpree@hotmail.com', 'yyxiong@gmail.com', 'yyxiong@tetx.com1', 'yyxiong@tetx.com2', 'yyxiong@tetx1.com'))
group by hour(pay_time)
```



### 03， 具体指标解析：

| 页面显示指标               |                           | 所用的数据                   |
| -------------------------- | ------------------------- | ---------------------------- |
| Real Orders                |                           | real_order_number_1day       |
| Dresses                    | Dresses(本身)             | dress1day                    |
|                            | Bridesmaid Dresses        | dress_bridesmaid_real_1day   |
|                            | Wedding Dresses           | dress_bridal_real_1day       |
|                            | MOB Dresses               | dress_mob_real_1day          |
|                            | Special Occasion Dresses  | dress_sod_real_1day          |
|                            | Junior Bridesmaid Dresses | dress_jbd_real_1day          |
|                            | Flower Girl Dresses       | dress_fgd_real_1day          |
|                            |                           |                              |
| Accessories                | Accessories(本身)         | accessory_real_1day          |
|                            | Separates, Wraps, Sashes  | separate_wrap_sash_real_1day |
|                            | Wedding Veils             | veil_real_1day               |
|                            | Groomsmen Accessories     | groomsmen_real_1day          |
|                            |                           |                              |
| Samples                    |                           | sample1day                   |
|                            | Bridesmaid Dresses        | sampleBridesmaid1day         |
|                            | Wedding Dresses           | sampleBridal1day             |
|                            | MOB Dresses               | sampleMotherBridal1day       |
|                            | Wedding Veils             | veil_real_1day               |
|                            |                           |                              |
| Ready To Ship（Clearance） |                           | clearance_real_1day          |
|                            |                           |                              |



## 3，flash sale相关数据方案

### flash sale的销量

```mysql
select c01.dayTime as '日期', c01.goods_id, c01.cat_id, sum(c01.dress_count) as '销量', 
if(c01.country=3844, 'CA', 'US') as country
from 
(
select b01.user_id, b01.order_id, b01.goods_id, b01.cat_id, b01.dress_count, b02.country, b01.dayTime
from

(select a02.user_id, a02.order_id, a01.goods_id, a03.cat_id, sum(a01.goods_number) as dress_count, day(a02.pay_time) as dayTime from 
order_goods as a01
left join 
order_info as a02
on a01.order_id = a02.order_id
left join 
goods as a03
on a01.goods_id = a03.goods_id
where a02.pay_time >= '2019-09-10 00:00:00' and a02.pay_time < '2019-09-18 00:00:00' 
and a02.pay_status=2 and a02.project_name = 'Azazie'
and (a02.email not like '%@tetx.com' and a02.email not like '%@i9i8.com' 
	and a02.email not like '%@qq.com' and a02.email not like '%@163.com' 
	and a02.email not like '%@126.com' and a02.email not like '%@jjshouse.com' 
	and a02.email not like '%@jenjenhouse.com' 
	and a02.email not like '%@azazie.com')
and a01.goods_id in 
(1003230,
1003231)
group by a02.user_id, a02.order_id, a01.goods_id, a03.cat_id, day(a02.pay_time)) as b01

join

(select user_id, country from user_address where address_type = 'SHIPPING' and is_default = 1 and country in (3859,3844)) as b02
on b01.user_id = b02.user_id) as c01
group by c01.goods_id, c01.cat_id, c01.country, c01.dayTime
order by c01.dayTime, c01.goods_id, c01.cat_id, c01.country
```



