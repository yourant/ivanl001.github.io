[toc]

> 本文档具体阐述azazie项目在商品相关属性更改后，如何增量更新到索引中。
>
> 如果有问题，可以直接查看相关条目即可



## 1, 更新流程: db -> maxwell -> sqs -> tinker-java -> es

* 01, 首先maxwell通过binlog进行监听azazie业务数据库中指定表变化

* 02, maxwell监听到后，发送消息到amazon sqs队列中去，队列为FIFO队列，保证顺序

* 03, 代码中消费amazon sqs消息，如果包含goods_id，且属于监听表的范围之内，则从数据库读取所需要的信息，组成索引信息，以goods_id为es的id，动态更新到es中



## 2, maxwell相关

* maxwell由运维同事施大康进行部署，直接监听azazie数据库变动，然后配置sqs队列，可方便的写入到我们的amazon 的sqs队列中去

* 01, maxwell监听表：

  >  goods 
  >  goods_languages 
  >  goods_project 
  >  goods_display_order (该表因为数据量巨大，所以放弃监听)
  >  color_family 
  >  goods_name_history 
  >  goods_color_style 
  >  goods_price_currency_map 
  >  goods 
  >  goods_attr
  >  attribute
  >  attribute_languages 
  >  goods_category 
  >  goods_display_order 
  >  goods 
  >  goods_attr 
  >  attribute 
  >  attribute_languages 
  >  category_languages 
  >  goods 
  >  goods_project 
  >  goods_gallery 
  >  style_languages 
  >   unfamiliar_color 
  >   search_redirect_url 



## 3, sqs相关

* sqs是amazon的队列服务， Amazon Simple Queue Service，  可以实现应用程序解耦,以及可靠性保证 
* sqs的队列类型有两种：standard和FIFO
* 我们在生产中使用的队列类型是FIFO队列，可保证先进先出



## 4, tinker-java(更新es代码)相关

> tinker是优化组优化代码的一个别名， tinker是php优化代码项目，tinker-java是java优化代码项目

### 4.1, 代码中表监听控制

* 虽然maxwell中已经对表进行限制，不过方便后续可能会有其他地方使用该方法，也为了数据安全，代码中也进行了表的限制

* 增量更新es监听的表如下(比maxwell少了几张，因为考虑到后续全量更新，所以maxwell中监听的表更多)

  ```java
  goods,goods_languages,goods_project,goods_name_history,goods_color_style,goods_price_currency_map,goods_attr,goods_category,attribute_languages,category_languages,goods_gallery
  ```



### 4.2, 代码中其他优化

> 01, 首先在消费sqs消息的时候进行批量获取，消息条数最大批次5条, 最大等待时间3s
>
> 02, 接受到sqs消息时候，并不是每次接受到goods_id就查询数据库，而是需要满足两个条件其中任何一个才会查询数据库更新es：
>
> 第一：和上次更新时间差大于30s(如果需要动态更改，可写入数据库中)
>
> 第二:  商品goods_id去重后总个数超过5个(如果需要动态更改，可写入数据库中)



### 4.3, tinker-java架构介绍

tinker-java是java项目：

使用springboot进行快速开发

数据库连接池使用hikari

架构分层：dao是数据连接层， service是服务层，controller是控制层。

文件夹：

dao文件夹是dao层的代码

service文件夹是service层代码

controller文件夹是controller层代码

app文件夹是不同的外部应用包，包括：amazon.sqs, elasticsearch, sql三个包和AppStarter， 其中AppStarter类是外部应用的启动类， 内部开启一个新的线程进行遍历获取sqs消息，然后进行逻辑处理

model是模型类， 在从数据库获取数据的时候都是直接动态转换模型方便后续调用

mapper是mybatis层，属于dao层，后续会替代dao层成为数据库连接层，目前等待全量更新的时候进行处理。



### 4.4, 手动增量更新

* 如果因为某些原因需要手动更新某一条数据，或者某些goods_id的索引，可以调用如下api:
* (调用前需要联系运维开通必要的端口号和ip)
* http://127.0.0.1:8080/es/updata?goodsId=1000001&languageId=1&project=Azazie



