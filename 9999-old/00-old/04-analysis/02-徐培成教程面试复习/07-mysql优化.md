- 01, 数据量小，通过所谓的LAMP(linux, apache, mysql, php)搭建即可满足数千用户的需求
- 02, 如果数据库压力大时，需要重新审视数据库设计是否合理。主键索引等等使用是否合适
- 03, 下面还不行，进行主从分离，主服务器进行写入操作，从服务器进行读取
- 04, 压力更大时候，增减缓存，如Memcached
- 05，强服务器，进行垂直扩容
- 06, 更大压力时候，采用逆范式化存储结构，停用存储过程
- 07, 最后分区
- 08, 还不行，就只能nosql了

