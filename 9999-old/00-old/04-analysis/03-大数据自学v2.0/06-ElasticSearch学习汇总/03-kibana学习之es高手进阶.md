[toc]



## 1, 精准匹配

## 基于bool组合多个filter搜索数据

### 1.1, term学习：sql语句转DSL

* select * from kibana_sample_data_ecommerce where sku contains ("ZO0549605496")
* select * from kibana_sample_data_ecommerce where customer_full_name in ('Eddie Underwood', 'Boris Maldonado')
* select * from kibana_sample_data_ecommerce where customer_full_name != 'Eddie Weber' and (customer_id = 38 or order_id = 727462)
* select * from kibana_sample_data_ecommerce where customer_first_name = 'Eddie' or (customer_first_name != 'Hardy' and customer_id = 20)

```shell
# select * from kibana_sample_data_ecommerce where sku contains ("ZO0549605496")
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "terms": {
      "sku": [
        "ZO0549605496"
      ]
    }
  }
}

# select * from kibana_sample_data_ecommerce where customer_full_name in ('Eddie Underwood', 'Boris Maldonado')
# 其实这里的must改成should也可以
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "terms": {"customer_full_name.keyword": ["Eddie Underwood","Boris Maldonado"]}
      },
      "boost": 1.2
    }
  },
  "from": 0, 
  "size": 100
}

GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must":{
            "terms": {
            "customer_full_name.keyword": [
              "Eddie Underwood",
              "Boris Maldonado"
            ]
            }
          }
        }
      },
      "boost": 1.2
    }
  },
  "from": 0, 
  "size": 100
}

# select * from kibana_sample_data_ecommerce where customer_full_name != 'Eddie Weber' and (customer_id = 38 or order_id = 727462)
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {"bool" : {
              "must_not" : {"term":{"customer_full_name.keyword":"Eddie Weber"}}
            }},
            {"bool":{
              "should":[
                {"term":{"customer_id":38}},
                {"term":{"order_id":727462}}
                ]
            }}
            ]
        }
      },
      "boost": 1.2
    }
  },
  "from": 0,
  "size": 100
}

# select * from kibana_sample_data_ecommerce where customer_first_name = 'Eddie' or (customer_first_name != 'Hardy' and customer_id = 20)
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "should" : [
            {"term": {"customer_first_name.keyword":"Eddie"}},
            {"bool" : {
              "must":[
                {"term": {"customer_id":20}},
                {"bool":{
                  "must_not":{"term": {"customer_last_name.keyword":"Hardy"}}
                }}
                ]
            }}
            ]
        }
      },
      "boost": 1.2
    }
  },
  "from": 0, 
  "size": 110
}
```





### 1.2, range学习：范围range

可以看到:

regexp， term， range都可以放到must里面

```shell
GET nginxaccesslog-20200314/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must":[
            {"regexp": {"url.keyword": ".*/v1/auth/login.*"}},
            {"term": {"response_code.keyword": "200"}},
            {"range":{"@timestamp": {"gte": "2020-03-14T10:00:00.000Z", "lte": "2020-03-14T16:02:00.000Z"}}}
          ]
        }
      },
      "boost": 1.2
    }
  },
  "from": 0,
  "size": 20
}
```



### 1.3, exgexp学习

```shell
GET nginxaccesslog-20200314/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must":[
            {"regexp": {"url.keyword": ".*/v1/auth/login.*"}},
            {"term": {"response_code.keyword": "200"}},
            {"range":{"@timestamp": {"gte": "2020-03-14T10:00:00.000Z", "lte": "2020-03-14T16:02:00.000Z"}}}
          ]
        }
      },
      "boost": 1.2
    }
  },
  "from": 0,
  "size": 20
}
```





## 2, 全文搜索匹配

### 2.1, 简单匹配

```shell
# select * from kibana_sample_data_ecommerce where customer_full_name like '%Eddie%'
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "customer_full_name": "Eddie"
    }
  }
}
```





### 2.2, 分词匹配

```shell
# customer_full_name中至少包含Underwood， Eddie， 或者Elyssa中的两个
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "match": {
      "customer_full_name": {
        "query": "Underwood Eddie Elyssa America",
        "minimum_should_match": "50%"
      }
    }
  }
}
# 上面的需求也可以使用match变相实现
GET /kibana_sample_data_ecommerce/_search
{
  "query": {
    "bool": {
      "minimum_should_match": 2, 
      "should": [
        {"match": {"customer_full_name": "Underwood"}},
        {"match": {"customer_full_name": "Eddie"}},
        {"match": {"customer_full_name": "Elyssa"}},
        {"match": {"customer_full_name": "America"}}
      ]
    }
  }
}
```



### 2.3,  dis_max实现best field 

* 多字段搜索相同的内容，尽量把一个字段包含多个内容的数据放在前面，而不是把多个字段都匹配到少量内容的数据放在前面
* 例如想搜索标题或者文章中包含ivanl001 is the king of world, 那么我是想找文章或者标题中包含最多ivanl001 is the king of world的文本，而不是文本或者标题中分别包含少量局部信息的那些。

```shell
# 如果是这样的话， 一般是所有字段都有匹配的分数较高，而部分字段匹配全部内容另外一个字段无匹配的分数相对较低
GET /forum/_search
{
  "query": {
    "bool": {
      "should": [
        {"match": {
          "title": "java solution"
        }},
        {"match": {
          "content": "java solution"
        }}
      ]
    }
  }
}

# 这里好像并不管用
GET /forum/_search
{
  "query": {
    "dis_max": {
      "queries": [
        {"match": {
          "title": "java solution"
        }},
        {"match": {
          "content": "solution java"
        }}]
    }
  }
}
```



## 3, 聚合操作

* group by操作

```shell
# 每个演员参演电影
# select actor.name, count(1) group by actor.name
GET /movie_0216/_search
{
  "aggs": {
    "groupByActorName": {
      "terms": {
        "field": "actorList.name.keyword",
        "size": 10000
      }
    }
  }
}


# 每个演员参演电影的平均分
# select actor.name, avg(doubanScore) group by actor.name
GET /movie_0216/_search
{
  "aggs": {
    "groupByActorName": {
      "terms": {
        "field": "actorList.name.keyword",
        "size": 10000
      },
      "aggs": {
        "avg_score": {
          "avg": {
            "field": "doubanScore"
          }
        }
      }
    }
  }
}




# 每个演员参演电影的平均分,再根据平均分进行排序
# select actor.name, avg(doubanScore) group by actor.name order by avg(doubanScore) desc
GET /movie_0216/_search
{
  "aggs": {
    "groupByActorName": {
      "terms": {
        "field": "actorList.name.keyword",
        "order": {
          "avg_score": "desc"
        }, 
        "size": 10000
      },
      "aggs": {
        "avg_score": {
          "avg": {
            "field": "doubanScore"
          }
        }
      }
    }
  }
}



# 每个演员参演电影的所有数据，包括：数量，最小值，最大值，平均值，加和值
# select actor.name, avg(doubanScore) group by actor.name order by avg(doubanScore) desc
GET /movie_0216/_search
{
  "aggs": {
    "groupByActorName": {
      "terms": {
        "field": "actorList.name.keyword", 
        "size": 10000
      },
      "aggs": {
        "avg_score": {
          "stats": {
            "field": "doubanScore"
          }
        }
      }
    }
  }
}
```

