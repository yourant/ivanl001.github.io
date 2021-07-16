

[toc]



## 1, 触发器基础

增加订单减少库存触发器， new代表新行

```mysql
delimiter $

create trigger t1
after 
insert 
on ord
for each row
begin 
update goods set num=num-new.much where gid=new.gid;
end$

```



显示所有触发器(因为上面把分号定义为$, 所以这里用$表示结束)

```mysql
show triggers;
```



删除触发器

```mysql
drop trigger t1;
```



删除订单增加库存触发器, old代表旧行

```mysql
delimiter $
create trigger t2
after 
delete
on ord
for each row
begin
update goods set num=num+old.much where gid=old.gid;
end$
```





更新订单(那么需要从库存中减去(订单的new数量-订单old数量))

更新订单数量逻辑可以理解成先删除原有订单，在增加新的订单，所以在库存上先加上老订单的数量，然后在减去新订单的数量

```mysql
delimiter $
create trigger t3
after 
update
on ord
for each row
begin
update goods set num=num+old.much-new.much where gid=old.gid;
end$
```





## 2, 触发器中的after和before

使用before可以判断库存是否大于订单量，如果大于，那么就可以做一定处理

如果使用after就不行

这个和t1触发器都是对order监听insert操作的触发器，所以如果添加这个，需要先删除前面的t2触发器哈

```mysql
delimiter $

create trigger t4
before
insert 
on ord

for each row
begin 

declare
the_num int;
select num into the_num from goods where gid=new.gid;

if new.much > the_num then 
set new.much = the_num;
end if;

update goods set num=num-new.much where gid=new.gid;
end$
```







