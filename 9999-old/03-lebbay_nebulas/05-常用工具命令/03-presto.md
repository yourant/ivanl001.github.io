

# presto时间相关的函数

> https://www.jianshu.com/p/786bfe77b8cb



```sql
select product_line, count(1)
from bl_and_sd.dws_first_paid_order as a 
-- presto格式转换
where format_datetime(first_pay_time, 'yyyy-MM-dd') = '2021-07-10'
group by product_line
```

