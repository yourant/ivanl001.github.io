[toc]



## 1, admin

### 01, es查看基本信息

```shell
############################-------es查看基本信息-------############################
# 查看es集群状态
GET _cat/health?v

# 查看es索引状态
# green： 每个索引的primary shard 和 replica shard 都是active状态
# yellow: 每个索引的primary shard 是active转台，但是replica shard 不是active状态，处于不可用状态
# red: 部分索引的primary shard是非active状态，部分索引数据丢失
GET /_cat/indices?v

# 查看每个shards的情况
GET /_cat/shards?v

# 查看所有的别名信息, 只会显示有别名的
GET /_cat/aliases

# 查看节点任务线程池相关的信息
GET /_cat/master?v
GET /_cat/nodes?v
GET /_cat/nodeattrs?v
GET /_cat/pending_tasks
GET /_cat/count?v
GET /_cat/thread_pool?v


# 查看集群的健康状况(json)
GET /_cluster/health
# 查看集群的状态,几个节点，每个节点的信息，以及所有索引等(json)
GET /_cluster/state
# 查看所有的别名信息(json)
GET /_alias


# 获取具体某个索引的的基本信息，包括aliases， mappings和settings
GET /kibana_sample_data_ecommerce

# 获取索引的alias
GET /kibana_sample_data_ecommerce/_alias
# 获取索引的mappings(后面可以指定type)
GET /kibana_sample_data_ecommerce/_mapping
GET /kibana_sample_data_ecommerce/_mapping/_doc
# 获取索引的settings
GET /kibana_sample_data_ecommerce/_settings

# 查看索引模版！！！！！！
GET /_cat/templates
```



### 02, es创建删除索引及配置mapping别名等

```shell

############################-------es创建删除索引-------############################

# 简单创建索引
PUT /ivanl001

# 创建索引，并指定相关设置(最好指定shards个数和副本个数), 别名和mapping映射
PUT ivanl001
{
  "aliases": {
    "ivanlsIndex": {}
  }, 
  "mappings": {
    "orders": {
      "properties" : {
        "order_id" : {
          "type" : "long"
        },
        "customer_name" : {
          "type" : "text",
          "analyzer":"ik_smart"
          "fields" : {
            "keyword" : {
              "type" : "keyword"
            }
          }
        },
        "money" : {
          "type" : "long"
        }
      }
    }
  }, 
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }
}

# 删除索引
DELETE ivanl001

# 删除索引(加逗号可以一次删除多个索引)
DELETE ivanl001,ivanl002,ivanl003

```



### 03, es更改配置别名mapping等

```shell
############################-------es更改配置别名mapping等-------############################
# 给某个索引单独添加别名，一个索引可以添加多个别名
# 多个别名是原子性操作
POST /_aliases
{
  "actions" : [
      { "add" : { "index" : "ivanl001", "alias" : "ivanlsIndex" } }
  ]
}

# 删除某个索引的某个别名
POST /_aliases
{
  "actions" : [
      { "remove" : { "index" : "ivanl001", "alias" : "ivanlsIndex" } }
  ]
}

# ------------------------------------------------------------------------------------------------
# 这个是原子性操作哈，不会造成中间来数据的有问题
POST /_aliases
{
  "actions" : [
      { "remove": { "index" : "movie_chn0217", "alias" : "the_movie" } },
      { "add" : { "index" : "movie_0216", "alias" : "the_movie" } }
  ]
}


# 给某个索引的某个type更改mapping
# 这里用post方法也行
PUT /ivanl001/_mapping/orders
{
  "properties" : {
    "order_id" : {
      "type" : "long"
    },
    "customer_name" : {
      "type" : "text",
      "fields" : {
        "keyword" : {
          "type" : "keyword"
        }
      }
    },
    "money" : {
      "type" : "long"
    },
    "count" : {
      "type": "long"
    }
  }
}

# 更改某个索引的setting部分
PUT /ivanl001/_settings
{
  "settings": {
    "number_of_replicas": 2
  }
}
```



## 2,  CRUD相关

### 01, 插入数据

#### 单条插入

put必须指定id，因为put是幂等性操作

post可以不必制定id，指定id也可以，post不是幂等操作

```shell
# 不指定id，自动生成id
POST /ivanl001/orders
{
  "order_id" : 0,
  "customer_name" : "ivanl bool", 
  "money" : 100,
  "count" : 1
}

# 指定id
POST /ivanl001/orders/1
{
  "order_id" : 1,
  "customer_name" : "ivanl bool", 
  "money" : 102,
  "count" : 3
}
```

#### bulk批量操作(可以是创建即插入，删除，更新等操作)

```shell
# delete只需要一条
# index是创建索引，如果已经存在，会覆盖，并且版本号会加1
# create是创建索引，如果已经存在，会报错版本冲突
# update是更新索引，需要指定doc字段，跟index类似，不过是局部更新，可以只给定需要更新的字段
POST /_bulk
{"delete": {"_index":"ivanl002", "_type":"person", "_id": "1"}}
{"delete": {"_index":"ivanl002", "_type":"person", "_id": "297-2XABqqLoAG5ZZFUT"}}
{"index":{"_index":"ivanl002", "_type":"person", "_id": "1"}}
{"first_name":"ivanl003","age":100,"about":"I aaaaa","interests": ["abc","def"]}
{"index":{"_index":"ivanl002", "_type":"person", "_id": "2"}}
{"first_name":"ivanl002","age":20,"about":"I am ivanl002","interests": ["love","heihei"]}
{"create":{"_index":"ivanl002", "_type":"person", "_id": "3"}}
{"first_name":"ivanl002","age":20,"about":"I am ivanl002","interests": ["love","heihei"]}
{"update":{"_index":"ivanl002", "_type":"person", "_id": "3"}}
{"doc":{"age":3000,"about":"I am ivanl00433"}}

# 当然也可以使用如下方式:

```



### 02, 更新数据

#### 更新方式1: 替换指定文档

#### 更新方式2: 直接更新指定文档,  "doc"是系统要求字段

```shell
# 对于已经存在的数据，不进行操作。如果不存在，进行创建
# 如果没有_create，则已经存在会进行替换
# /index/type/id/_create
# 如果没有_create，则已经存在会进行替换，也就是全量更新。
PUT /ivanl001/person/1/_create
{"first_name" : "bin",
    "age" : 33,
    "about" : "I love to go rock climbing",
    "interests" : [
      "sports",
      "music"
    ]
}

# 更新方式1: 替换指定文档
PUT ivanl002/person/1
{
 "first_name" : "bin--0004",
 "age" : 33,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}

# 更新方式2: 直接更新指定文档,  "doc"是系统要求字段
POST ivanl002/person/1/_update
{
 "doc":{
    "first_name" : "ivanl0007"
 }
}
```



### 03, 数据查询检索后续讲



## 3, 不同的数据检索方式

### 01, query string search

```shell
# 搜索方式1
# query string search
GET /ivanl001/_search
GET /ivanl001/person/_search
GET /ivanl001/person/_search?q=age:33
# 这里是包含，只要包含就可以
GET /ivanl001/person/_search?q=interests:sports

GET /kibana_sample_data_ecommerce/_search
# 指定查询(包含)，并排序
GET /kibana_sample_data_ecommerce/_search?q=customer_full_name:Simpson&sort=taxless_total_price:desc
# _source指定返回的字段
GET /kibana_sample_data_ecommerce/_search?q=customer_full_name:Simpson&sort=taxless_total_price:desc&_source=customer_full_name,taxless_total_price

# 如果customer_first_name是字符串，那么下面这四句等价
GET /kibana_sample_data_ecommerce/_doc/_search?q=customer_first_name:Eddie
GET /kibana_sample_data_ecommerce/_doc/_search?q=+customer_first_name:Eddie
GET /kibana_sample_data_ecommerce/_doc/_search?q=customer_first_name=Eddie
GET /kibana_sample_data_ecommerce/_doc/_search?q=+customer_first_name=Eddie

# 如果order_id不是字符串，那么只能用如下两种方式匹配，不能用等号，只能用:
GET /kibana_sample_data_ecommerce/_doc/_search?q=customer_first_name:Eddie
GET /kibana_sample_data_ecommerce/_doc/_search?q=+customer_first_name:Eddie

# ?????问题：多个参数如何组装？逗号好像不好用？？？


# query string(+符号和-符号)
# 无任何符号的时候代表匹配，也就是和+等效, 表示只要包含Eddie即可
GET /kibana_sample_data_ecommerce/_doc/_search?q=customer_first_name=Eddie

# +符号代表匹配，必须能匹配到才行，表示只要包含Eddie即可
GET /kibana_sample_data_ecommerce/_doc/_search?q=+customer_first_name=Eddie

# -符号代表排除，必须能匹配不到对应的内容才行，表示只要不包含Eddie即可
GET /kibana_sample_data_ecommerce/_doc/_search?q=-customer_first_name=Eddie


# q=Eddie，这里是匹配所有字段
# 当然这个底层并不是直接匹配所有字段，而是在创建所以的时候，es会把多个字段串联起来形成一个metadata原数据，这样子在搜素所有字段的时候其实只需要匹配metadata原数据这一个字段就可以了
GET /kibana_sample_data_ecommerce/_doc/_search?q=Eddie
```



### 02, query DSL (简单介绍)

>  query DSL和query string的区别

* query DSL是把参数封装在body中
* query string就是直接封装在请求中

```shell
# 分页搜索， query string方式
GET /kibana_sample_data_ecommerce/_doc/_search?from=0&size=5

# 有些浏览器不支持get带请求体，那么更换成post方式即可， query DSL方式
GET /kibana_sample_data_ecommerce/_doc/_search
{
  "from" : 0,
  "size" : 5
}
```



#### 01, query, sort, from, size, query中match是全文匹配

* 匹配分词后包含的数据

> query里面主要是match匹配或者match_phrase匹配，通过可以同过bool条件进行组合match匹配

```shell
# 搜索方式2
# query DSL： domain specified language
# 查询所有
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match_all": {}
  }
}

# 查询用户名中包含Gwen或者Butler而且按照无税价格降序
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "customer_full_name": "Gwen Butler"
    }
  },
  "sort": [
    {
      "taxless_total_price": {
        "order": "desc"
      }
    }
  ]
}

# 分页查询, 每页显示10条，显示第二页
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "customer_full_name": "Gwen Butler"
    }
  },
  "sort": [
    {
      "taxless_total_price": {
        "order": "desc"
      }
    }
  ],
  "from": 10,
  "size": 10
}

# 执行查询的商品的具体信息,指定需要的字段，_source
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "customer_full_name": "Gwen Butler"
    }
  },
  "sort": [
    {
      "taxless_total_price": {
        "order": "desc"
      }
    }
  ],
  "_source": ["customer_full_name", "taxless_total_price", "products._id", "products.product_name"], 
  "from": 10,
  "size": 10
}
```



#### 02, query, sort, from, size, query中match_phrase是短语搜索

* 匹配包含的数据

```shell
# 这个地方是完整匹配，搜索的内容需要完全匹配，不会分词
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match_phrase": {
      "customer_full_name": "Gwen Butler"
    }
  }
}
```



#### 03, query, sort, from, size, query中term是词条匹配

* 这个里面只能匹配keyword，也就是不分词的内容，一旦分词就匹配不出任何内容

```shell
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "term": {
      "customer_full_name.keyword": {
        "value": "Robert Weber"
      }
    }
  }
}

```



#### 04, 高亮搜索

```shell
# 搜索方式6
# 高亮搜索
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match_phrase": {
      "customer_full_name": "Gwen Butler"
    }
  },
  "highlight": {
    "fields": {
      "customer_full_name": {}
    }
  }
}


# 也可以自定义高亮显示规则
GET /movie_0216/_search
{
  "query": {
    "match": {
      "name": "red"
    }
  },
  "highlight": {
    "fields": {
      "name": {
        "pre_tags": "<span color='red'>",
        "post_tags": "</span>"
      }
    }
  }
}

```



#### 05, fuzzy（容错匹配）

```shell
# 主要是在英文中， 字母拼错可以有一定程度的纠正回来，
# 中文里好像不太好使
GET /movie_0216/_search
{
  "query": {
    "fuzzy": {
      "name": "rad"
    }
  }
}
```







## 4, 定制组合查询排序

### 01, bool中的四个判断：must, must_not, should, filter

> bool里面可以放四个：
>
> ​	must, must_not, should, filter  
>
> ​    ------->  其中filter中不记分，只是过滤
>
> 这几个相当于是控制逻辑关系，相当于是and or

```shell
# 匹配所有
GET /kibana_sample_data_ecommerce/_doc/_search
{
  "query" : {
    "match_all": {}
  }
}

# 所以must, must_not, should的用法大致如下:
# 如果全部是and连接,并且都是=，那么所有的都封装在must的[]中即可
# 如果全部是and连接，并且都是!=,那么全部都封装在must_not的[]中即可
# 如果全部是or连接, 那么全部封装在should中即可

# 全部是and连接: customer_first_name=Eddie and customer_last_name=Weber
# 3
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"customer_first_name.keyword": "Eddie"}},
        {"match": {"customer_last_name.keyword": "Weber"}}
      ]
    }
  }
}


# 全部是and连接: customer_first_name!=Eddie and customer_last_name!=Weber
# 4560
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must_not": [
        {"match": {"customer_first_name.keyword": "Eddie"}},
        {"match": {"customer_last_name.keyword": "Weber"}}
      ]
    }
  }
}

# 全部是or连接: customer_first_name=Eddie or customer_last_name=Weber
# 115
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "should": [
        {"match": {"customer_first_name.keyword": "Eddie"}},
        {"match": {"customer_last_name.keyword": "Weber"}}
      ]
    }
  }
}

# 如果是复杂条件查询
# customer_first_name=Eddie and ("customer_last_name"="Underwood" or "customer_last_name"="Weber")
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"customer_first_name.keyword": "Eddie"}},
        {
          "bool": {
            "should": [
              {"match": {"customer_last_name.keyword": "Underwood"}},
              {"match": {"customer_last_name.keyword": "Weber"}}
            ]
          }
        }
      ]
    }
  }
}

# 注意：
# 如果是must和must_not放在一起，因为都是肯定，所以相当于两个里面的条件都需要满足
# 下面的逻辑是:customer_first_name=Eddie and customer_last_name!=Underwood
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": 
          {
            "customer_first_name.keyword": "Eddie"
          }
        }
      ],
      "must_not": [
        {"match": {
          "customer_last_name.keyword": "Underwood"
        }}
      ]
    }
  }
}

# 但是如果must must_not和should处于同一级的情况下，should内部的条件相当于没有
# 比如说下面should中的条件就是无效的
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"customer_first_name.keyword": "Eddie"}}
      ],
      "should": [
        {"match": {"customer_last_name.keyword": "Underwood"}}
      ]
    }
  }
}
```



### 02, query中的三个维度的查询:term, match_phrase, match

> term是最小维度， 表示文档中必须有这个词条(字段)，然后词条匹配才算匹配上(字段内容完全相同)
>
> match_phrase是term高一级的维度，表示词组，把该词组当作一个整体，必须要有等于或者包含这个词组的才算匹配上
>
> match则是更高一级的维度，表示会把匹配的数据完全分词打散，按照分词进行查询到就算匹配上

```shell
# 如下几个查询等价
# 因为使用的是customer_full_name.keyword匹配，这个keyword是不分词的，所以使用term, match还是match_phrase都是一样效果
GET /kibana_sample_data_ecommerce/_search
{"query": {
    "bool": {
      "must": [
        {"term": {"customer_full_name.keyword": "Eddie Weber"}}
      ]
    }
  }
}

GET /kibana_sample_data_ecommerce/_search
{"query": {
    "bool": {
      "must": [
        {"match_phrase": {"customer_full_name.keyword": "Eddie Weber"}}
      ]
    }
  }
}


GET /kibana_sample_data_ecommerce/_search
{"query": {
    "bool": {
      "must": [
        {"match": {"customer_full_name.keyword": "Eddie Weber"}}
      ]
    }
  }
}

# 但是如果使用customer_full_name匹配的话，
# term是匹配不出来的。因为customer_full_name被分词了，不是一个term了，就查不出来了。
# 只有match_phrase依然是整个词进行匹配, 但是会匹配出所有包含被匹配内容的数据，比如Eddie Weber Bool
# match是分词后匹配的，会匹配出被匹配内容分词后， 比如：Robert Weber
GET /kibana_sample_data_ecommerce/_search
{"query": {
    "bool": {
      "must": [
        {"term": {"customer_full_name": "Eddie Weber"}}
      ]
    }
  }
}

GET /kibana_sample_data_ecommerce/_search
{"query": {
    "bool": {
      "must": [
        {"match_phrase": {"customer_full_name": "Eddie Weber"}}
      ]
    }
  }
}


GET /kibana_sample_data_ecommerce/_search
{"query": {
    "bool": {
      "must": [
        {"match": {"customer_full_name": "Eddie Weber"}}
      ]
    }
  }
}
```











### 03, term查询的几种方式

```shell
# term query第一种方式 {"term": {"customer_full_name.keyword": "Eddie Weber"}}
GET /kibana_sample_data_ecommerce/_search
{"query": {
    "bool": {
      "must": [
        {"term": {"customer_full_name.keyword": "Eddie Weber"}}
      ]
    }
  }
}

# term query第一种方式 { "term": { "customer_full_name.keyword" : { "value": "Eddie Weber" }}
GET /kibana_sample_data_ecommerce/_search
{"query": {
    "term": {
      "customer_full_name.keyword": {
        "value": "Eddie Weber"
      }
    }
  }
}

# term query第一种方式 {"terms": {"customer_full_name.keyword": ["Eddie Weber","Eddie Underwood"]}
GET /kibana_sample_data_ecommerce/_search
{"query": {
    "terms": {
      "customer_full_name.keyword": [
        "Eddie Weber",
        "Eddie Underwood"
      ]
    }
  }
}
```



### 04, filter相关

```shell
# 01, filter中的过滤条件对评分无影响，但是如果没有filter过滤，则会对评分造成影响， 放在filter中则对于评分无影响
# total_quantity直接过滤会产生评分影响
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "range": {
      "total_quantity": {
        "gte": 3
      }
    }
  }
}

# 02, 通过filter过滤效果会高很多，不进行评分，是直接过滤掉
# 这里total_quantity是过滤中的一个字段，所以对于数据不会有影响
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"order_date": "2020-03-22"}}
      ],
      "filter": {
        "range": {
          "total_quantity": {
            "gte": 2
          }
        }
      }
    }
  },
  "from": 0,
  "size": 200
}

# 03, 如果只需要过滤的时候，可能需要特殊处理
# constant_score
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "total_quantity": {
            "gte": 2
          }
        }
      }
    }
  }
}
```



### 04，post_filter和filter(是否先匹配后过滤)

```shell
# 先查询，后根据查询结果进行过滤
GET /movie_0216/_search
{
  "query": {
    "match": {
      "name": "red"
    }
  },
  "post_filter": {
    "term": {
      "actorList.name.keyword": "zhang han yu"
    }
  }
}

# 查询过滤一起
GET /movie_0216/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "name": "red"
        }}
      ],
      "filter": {
        "term": {
          "actorList.name.keyword": "zhang han yu"
        }
      }
    }
  }
}
```





### 05, multi_match: 在多个字段中进行匹配

```shell
# 01, 在多个字段中同时匹配
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "should": [
        {"multi_match": {
          "query": "Eddie",
          "fields": ["customer_first_name", "customer_full_name"]
        }}
      ]
    }
  }
}
```



### 06, query, sort, from, size, query中的sort

```shell
# 结果定制排序
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"order_date": "2020-03-22"}}
      ],
      "filter": {
        "range": {
          "total_quantity": {"gte": 2}
        }
      }
    }
  },
  "sort": [
    {"total_quantity": {"order": "desc"}}
  ], 
  "from": 0,
  "size": 200
}


# 排序
GET /movie_0216/_search
{
  "query": {
    "bool": {
    }
  },
  "post_filter": {
    "range": {
      "doubanScore": {
        "gte": 6,
        "lte": 9.4
      }
    }
  },
  "sort": [
    {
      "doubanScore": {
        "order": "asc"
      }
    }
  ]
}
```



> 如果对字符串进行排序，最好进行两次索引，需要一个全文字符串，用全文字符串进行排序
>
> 目前已经已经默认会对字符串进行两次索引了

```shell
{
    "type": "text",
    "fields": {
        "keyword": {
            "type": "keyword",
            "ignore_above": 256
        }
    }
}
```



### 07, range

```shell
# range可以直接放在查询同时过滤的bool的filter下面
GET /movie_0216/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "doubanScore": {
            "gte": 6,
            "lte": 8.4
          }
        }
      }
    }
  }
}

# range也可以放在和query同级的post_filter里面
GET /movie_0216/_search
{
  "query": {
    "bool": {
    }
  },
  "post_filter": {
    "range": {
      "doubanScore": {
        "gte": 6,
        "lte": 8.4
      }
    }
  }
}
```





## 5, 读取数据

### 00:  常规获取(可分页)

```shell
# 获取一个索引的所有数据
GET /kibana_sample_data_ecommerce/_search
# 获取一个索引下的指定type的所有数据
GET /kibana_sample_data_ecommerce/_doc/_search

# 注意：上面默认情况下是显示10条数据。可以通过from和size参数来获取需要的数据。
# 给定参数的方式有两种：query string search和query DSL： domain specified language, 这个后续会讲
# 分页搜索， query string方式
GET /kibana_sample_data_ecommerce/_doc/_search?from=0&size=5

# 这里可以看到，如果是对象内部数据，需要按照json方式获取
GET /company/employee/_search?q=address.country=china

# 有些浏览器不支持get带请求体，那么更换成post方式即可， query DSL方式
GET /kibana_sample_data_ecommerce/_doc/_search
{
  "from" : 0,
  "size" : 5
}
```



### 01: 单条获取

```shell
# 给定/indes/type/id方式获取单条数据
GET /kibana_sample_data_ecommerce/_doc/Z9gbx3ABqqLoAG5Z25uo
```



### 02: mget获取多条

```shell
# 0101----同一index，同一type下指定id
GET /kibana_sample_data_ecommerce/_doc/_mget
{
  "docs": [
    {"_id":"Z9gbx3ABqqLoAG5Z25uo"},
    {"_id":"aNgbx3ABqqLoAG5Z25uo"}
  ]
}

# 0101
GET /kibana_sample_data_ecommerce/_doc/_mget
{
  "ids":["Z9gbx3ABqqLoAG5Z25uo", "aNgbx3ABqqLoAG5Z25uo"]
}

# 0101----这里同时筛选出需要的字段
GET /kibana_sample_data_ecommerce/_doc/_mget
{
  "docs": [
    {
      "_id":"Z9gbx3ABqqLoAG5Z25uo", 
      "_source": ["customer_full_name", "taxless_total_price"] 
    },
    {
      "_id":"Z9gbx3ABqqLoAG5Z25uo", 
      "_source": ["customer_full_name", "taxless_total_price"]
    }
  ]
}

# 0102----同一index下不同type
GET /kibana_sample_data_ecommerce/_mget
{
  "docs" : [
    {"_type": "_doc", "_id": "Z9gbx3ABqqLoAG5Z25uo"},
    {"_type": "_doc", "_id": "aNgbx3ABqqLoAG5Z25uo"}
    ]
}

# 0103----不同index下不同type
GET /_mget
{
  "docs" : [
    {"_index": "kibana_sample_data_ecommerce", "_type": "_doc", "_id": "Z9gbx3ABqqLoAG5Z25uo"},
    {"_index": "kibana_sample_data_ecommerce", "_type": "_doc", "_id": "aNgbx3ABqqLoAG5Z25uo"}
    ]
}
```



### 03: scroll滚动获取

 ```shell
# 第一次请求设定scroll的基本方式排序窗口时长等
GET /kibana_sample_data_ecommerce/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": ["_doc"],
  "size": 1
}

# 之后每次取就之前通过上次的id获取即可
GET _search/scroll
{
  "scroll": "1m",
  "scroll_id": "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAmpsWeXI3RHJyX1RTcC1CcTUyaWF4cDM0UQ=="
}

# 取到最后一次就取不出任何结果了
{
  "_scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAgUWdEZuOVN0OUJRUk9ubzVEbTF6dEtpQQ==",
  "took" : 3,
  "timed_out" : false,
  "terminated_early" : true,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 4675,
    "max_score" : null,
    "hits" : [ ]
  }
}
 ```





## 6, es的分词器

### 01, es自带分词器分词效果简介

```shell
# Set the shape to semi-transparent by calling set_trans(5)

# standard analyzer(默认)
set, the, shape, to, semi, transparent, by, calling, set_trans, 5

#simple analyzer
set, the, shape, to, semi, transparent, by, calling, set, trans

# whitespace analyzer
Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

# language analyzer(特定语言分词器，下面这个是english分词器分词结果)
set, shape, semi, transpar, call, set_tran, 5
```



### 02, 查看分词结果

```shell
# 测试分词效果
GET _analyze
{
  "analyzer": "standard",
  "text": "ivanl001 is the king of world!"
}
```



### 03,自定义分词器并使用

* 参考：https://www.elastic.co/guide/cn/elasticsearch/guide/current/language-intro.html

```shell
# ----------------------------------------1,分词器的简单测试----------------------------------------
# 先删除
DELETE /my_index
# 自定义分词器， 这里是启用英文停用词，也就是介词，副词等都直接过滤掉
PUT /my_index
{
  "settings" : {
    "analysis": {
      "analyzer": {
        "es_std" : {
          "type": "standard",
          "stopwords": "_english_"
        } 
      }
    }
  }
}
GET /my_index/_settings
# 测试分词效果， 标准情况下是不删除停用词都
GET /my_index/_analyze
{
  "analyzer": "standard",
  "text": "ivanl001 is the king of world!"
}
# 使用分词器es_std的时候就启用了
GET /my_index/_analyze
{
  "analyzer": "es_std",
  "text": "ivanl001 is the king of world!"
}


# ----------------------------------------2,自定义自己的分词器----------------------------------------
DELETE /my_index

PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "&_to_and":{
          "type": "mapping",
          "mappings": ["& => and"]
        }
      },
      "filter": {
        "my_stopwords":{
          "type":"stop",
          "stopwords": ["the", "a"]
        }
      },
      "analyzer": {
        "custom_analyzer": {
          "type" : "custom",
          "char_filter": ["html_strip", "&_to_and"],
          "tokenizer" : "standard",
          "filter": ["lowercase", "my_stopwords"]
        }
      }
    }
  }
}

GET /my_index/_mapping

# 进行测试自定义分词器
GET /my_index/_analyze
{
  "text": "Ivanl001 is the king of world & the king of hahah<h1>zhang<h1>",
  "analyzer": "custom_analyzer"
}

# ---------------------------3,在mapping的时候使用自定义的分词器------------------------------

PUT /my_index/_mapping/my_type
{
  "properties": {
    "content" : {
      "type": "text",
      "analyzer": "custom_analyzer"
    }
  }
}

```



### 04, ik中文分词器

* 具体安装见安装文档

```shell
#### ik_smart演示
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "我是中国人"
}
# 分词结果
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国人",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}

#### ik_max_word演示
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "我是中国人"
}
# 分词结果
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国人",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "中国",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "国人",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}
```



### 05，自定义词库

```shell
1, 修改ik分词器配置文件
vim /usr/local/es/plugins/ik/config/IKAnalyzer.cfg.xml

<!--用户可以在这里配置远程扩展字典 -->
<!-- <entry key="remote_ext_dict">words_location</entry> -->
<entry key="remote_ext_dict">http://centos01:80/custom_dict/boolkeyword.txt</entry>

2, 
```





## 7, alias, setting, mapping及索引模版

### 01, 设置mapping

```shell
# 是否分词已经改变
# 如果分词type就是text，如果不分词type就是keyword
PUT /website
{
  "mappings": {
    "article":{
      "properties" :{
        "author_id": {
          "type" : "long"
        },
        "title": {
          "type" : "text",
          "analyzer":  "english"
        },
        "content": {
          "type": "text"
        },
        "post_date": {
          "type": "date"
        },
        "publisher_id": {
          "type": "keyword"
        }
      }
    }
  }
}

# 查看
GET /website/_mapping

# 只能添加不能更改
PUT /website/_mapping/article
{
  "properties" :{
    "tags": {
      "type" : "text"
    }
  }
}

# 测试，使用english分词器，所以没有a这个介词
GET /website/_analyze
{
  "field": "title",
  "text" : "ivanl00 a dog"
}

# 测试, 因为默认使用starndard分词器，所以会有a
GET /website/_analyze
{
  "field": "content",
  "text" : "ivanl00 a dog"
}

# 测试，publisher_id是keywords，所以不分词
GET /website/_analyze
{
  "field": "publisher_id",
  "text" : "ivanl00 a dog"
}
```



### 02，索引模版

> 索引模版：通过建立索引模版，可以再插入当天第一条数据的时候自动为当天建立索引，而不必每天都手动的创建索引

```shell
# 先创建一个模版索引
PUT _template/template_movie
{
  "index_patterns": ["movie_test*"],                  
  "settings": {                                               
    "number_of_shards": 1
  },
  "aliases" : { 
    "{index}-query": {},
    "movie_test-query":{}
  },
  "mappings": {                                          
"_doc": {
      "properties": {
        "id": {
          "type": "keyword"
        },
        "movie_name": {
          "type": "text",
          "analyzer": "ik_smart"
        }
      }
    }
  }
}

# 插入数据的时候可以直接插入，会根据模版索引进行自动创建索引
POST movie_test_20210101/_doc
{
  "id":"333",
  "name":"zhang3"
}

# 插入数据的时候可以直接插入，会根据模版索引进行自动创建索引
POST movie_test_20210102/_doc
{
  "id":"444",
  "name":"zhang4"
}

# 可以查到会自动新增两个索引
GET /_cat/indices?v

# 也会根据模版索引的设置自动新增别名
GET movie_test-query/_search
```







## 8, 其他碎知识

### 0.0, NRT(near real time)

* 近实时，两个意思，从写入数据到数据可以被搜索到有一个小延迟（大概1秒）；基于es执行搜索和分析可以达到秒级。



### 0.1, timeout

```shell
# 这个超时时间是每个lucence的超时时间，而不是es返回结果的那个超时时间。
# 所有设定这个超时时间为1ms，返回结果中的超时时间依然为3ms或者6ms。这个不冲突
GET twitter/tweet/_search?timeout=10ms
```



### 0.2, multi-index, multi-type搜索模式

```shell
# multi-index, multi-type搜索模式
# 这里是同时搜索多个index，多个type
GET /kibana_sample_data_ecommerce,kibana_sample_data_flights,twitter/_doc,tweet/_search

# 也可以使用通配符
GET /kibana_*/_doc/_search
# 也可以直接_all指定所有
GET /_all/tweet/_search
```



### 0.3, paging分页搜索

* 注意：如果有3个shards，搜索第2000-2100条数据的时候，需要取出每个shard上的2000-2100条数据，然后把6300条数据排序，取出其中的2000-2100条数据。
* 如果取数万条，而且分页比较靠后，就会是深度分页，所以这种深度分页又叫做deep paging， 这个过程其实是相对耗费内存和cpu的。大量数据返回到coordinator节点，所以耗费网络io, 需要把大量数据加载内存，耗费内存，排序，耗费cpu.

```shell
# 分页搜索
GET /kibana_sample_data_ecommerce/_doc/_search?from=0&size=5
```



### 0.4, es的核心数据类型

```shell
# 基础数据类型
string
byte, short, integer, long
float, double
boolean
date

# dynamic mapping
true or false. ---> boolean
123            ---> long
12.43          ---> double
2020-01-01     ---> date
"ivanl001"     ---> string/text
```



### 0.5, 对象数据

```shell
PUT /company/employee/1
{
  "address" : {
    "country" : "china",
    "province" : "shanghai",
    "city" : "shanghai"
  },
  "name": "ivanl001",
  "age": 27,
  "join_date": "2020-01-01"
}

GET /company/_mapping 
GET /company/employee/_search?q=china
# 这里可以看到，如果是对象内部数据，需要按照json方式获取
GET /company/employee/_search?q=address.country=china
```



### 0.6, explain

```shell
# exlain
GET /kibana_sample_data_ecommerce/_validate/query?explain
{
  "query": {
    "bool": {
      "must": [
        {"match": 
          {
            "customer_first_name.keyword": "Eddie"
          }
        }
      ],
      "must_not": [
        {"match": {
          "customer_last_name.keyword": "Underwood"
        }}
      ]
    }
  }
}


# +customer_first_name.keyword:Eddie -customer_last_name.keyword:Underwood
{
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "valid" : true,
  "explanations" : [
    {
      "index" : "kibana_sample_data_ecommerce",
      "valid" : true,
      "explanation" : "+customer_first_name.keyword:Eddie -customer_last_name.keyword:Underwood"
    }
  ]
}

```



## 9, es的高级应用







## 10, 内核知识

### 01, type

* 同一index，不同type的数据其实是存在一起的。如果两个字段完全不同的数据在一个type，在存储器中一个type的时候，另外一个type字段全部都是null，会造成大量的空间浪费



