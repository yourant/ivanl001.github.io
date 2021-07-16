```sql
-- 建表语句
create table track.ddid_user_data
	(report_date varchar not null, 
	product_line TINYINT  not null, 
	platform TINYINT  not null,
	country varchar not null, 
	channel varchar  not null, 
	site_id SMALLINT  not null, 
	goods_id INTEGER  not null, 
	ddid INTEGER  not null, 
	client_id varchar  not null, 
	count INTEGER, 
	CONSTRAINT PK PRIMARY KEY (report_date,product_line,platform,country,channel,site_id,goods_id,ddid,client_id)) 
	salt_buckets = 5;
	
	
-- 建表语句
create table track.ddid_user_data (report_date varchar not null, product_line TINYINT not null, platform TINYINT not null, country varchar not null, channel varchar not null, site_id SMALLINT not null, goods_id INTEGER not null, ddid INTEGER not null, client_id varchar not null, count INTEGER, CONSTRAINT PK PRIMARY KEY (report_date,product_line,platform,country,channel,site_id,goods_id,ddid,client_id)) salt_buckets = 5;

```

