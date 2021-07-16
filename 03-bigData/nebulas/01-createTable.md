[toc]

## sqoop导入

```shell
# -------------------------------------01-订单order_info导入-------------------------------------
#!/bin/bash

db_date=$1
echo $db_date

/opt/cloudera/parcels/CDH/bin/sqoop import \
--connect 'jdbc:mysql://blushmark-slave.cbb0nles4v8i.us-east-1.rds.amazonaws.com/blushmark?characterEncoding=UTF-8&autoReconnect=true&zeroDateTimeBehavior=convertToNull&serverTimezone=America/Los_Angeles' \
--username aztech1 \
--password 8m4UquCBkNYn \
--target-dir /user/hive/sqoop/tmp_bl_and_sd/order_info \
--table order_info  \
--split-by id  \
--hive-import  \
--hive-overwrite \
--hive-drop-import-delims \
--hive-database 'bl_and_sd'  \
--hive-table 'order_info' \
--fields-terminated-by '\001' \
--lines-terminated-by '\n' \
--where "(project_name = 'blushmark' or project_name = 'sisdress') and ((pay_time>= '$db_date' and pay_time<DATE_ADD(DATE('$db_date'), interval 1 day)) or (order_time>='$db_date' and order_time<DATE_ADD(DATE('$db_date'), interval 1 day)))" \
--hive-partition-key 'report_date' \
--hive-partition-value $db_date \
--m 1

# -------------------------------------02-订单商品order_goods导入-------------------------------------
#!/bin/bash

db_date=$1
echo $db_date

/opt/cloudera/parcels/CDH/bin/sqoop import \
--connect 'jdbc:mysql://blushmark-slave.cbb0nles4v8i.us-east-1.rds.amazonaws.com/blushmark?characterEncoding=UTF-8&autoReconnect=true&zeroDateTimeBehavior=convertToNull&serverTimezone=America/Los_Angeles' \
--username aztech1 \
--password 8m4UquCBkNYn \
--target-dir /user/hive/sqoop/tmp_bl_and_sd/order_goods \
--query "select og.* from order_goods as og join order_info as oi on og.order_id = oi.order_id where (project_name = 'blushmark' or project_name = 'sisdress') and ((pay_time>= '$db_date' and pay_time<DATE_ADD(DATE('$db_date'), interval 1 day)) or (order_time>='$db_date' and order_time<DATE_ADD(DATE('$db_date'), interval 1 day))) and \$CONDITIONS" \
--split-by og.rec_id \
--hive-import  \
--hive-overwrite \
--hive-drop-import-delims \
--hive-database 'bl_and_sd'  \
--hive-table 'order_goods' \
--fields-terminated-by '\001' \
--lines-terminated-by '\n' \
--hive-partition-key 'report_date' \
--hive-partition-value $db_date \
--m 1



# -------------------------------------03-订单拓展order_extras表导入-------------------------------------
#!/bin/bash

db_date=$1
echo $db_date

/opt/cloudera/parcels/CDH/bin/sqoop import \
--connect 'jdbc:mysql://blushmark-slave.cbb0nles4v8i.us-east-1.rds.amazonaws.com/blushmark?characterEncoding=UTF-8&autoReconnect=true&zeroDateTimeBehavior=convertToNull&serverTimezone=America/Los_Angeles' \
--username aztech1 \
--password 8m4UquCBkNYn \
--target-dir /user/hive/sqoop/tmp_bl_and_sd/order_extras \
--query "select oe.* from blushmark.order_info as oi join blushmark.order_extras as oe on oi.order_id = oe.order_id where project_name = 'blushmark' and ((pay_time>= '$db_date' and pay_time<DATE_ADD(DATE('$db_date'), interval 1 day)) or (order_time>='$db_date' and order_time<DATE_ADD(DATE('$db_date'), interval 1 day))) and \$CONDITIONS" \
--split-by oe.id \
--hive-import  \
--hive-overwrite \
--hive-drop-import-delims \
--hive-database 'bl_and_sd'  \
--hive-table 'order_extras' \
--fields-terminated-by '\001' \
--lines-terminated-by '\n' \
--hive-partition-key 'report_date' \
--hive-partition-value $db_date \
--m 1

# -------------------------------------03-订单商品拓展order_extras表导入-------------------------------------


```



