

![image-20200311114821877](assets/image-20200311114821877.png)





```mysql
# 查看es状态
# green： 每个索引的primary shard 和 replica shard 都是active状态
# yellow: 每个索引的primary shard 是active转台，但是replica shard 不是active状态，处于不可用状态
# red: 部分索引的primary shard是非active状态，部分索引数据丢失
GET _cat/health?v

# 查看所有的index
GET _cat/indices?v

# 获取es中的所有数据
GET _search
{
  "query": {
    "match_all": {}
  }
}

# 获取索引的的基本信息，包括aliases， mappings和settings
GET azazie

# 获取索引的所有内容
GET azazie/_search

# 获取索引中type内某一个具体id的内容
GET azazie/azazie/1000001

# 增加索引
PUT ivanl002

# 给某个索引添加具体的内容
POST ivanl002/person/2
{
 "first_name" : "bin",
 "age" : 33,
 "about" : "I love to go rock climbing",
 "interests": [ "sports", "music" ]
}

# 下面两个目前等同，因为6.0下一个index下只能有一个type
GET ivanl002/person/_search
GET ivanl002/_search

# 获取索引中type内某一个具体id的内容
GET ivanl002/person/1

# 查询匹配first_name="bin"的内容
GET /ivanl002/person/_search?q=first_name="bin"


# 创建索引的时候指定配置
PUT twitter
{
    "settings" : {
        "index" : {
            "number_of_shards" : 3, 
            "number_of_replicas" : 2 
        }
    }
}

# 删除索引
DELETE /test

# 
PUT test
{
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "properties" : {
                "field1" : { "type" : "text" }
            }
        }
    }
}


POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "ivanl002", "alias" : "alias1" } }
    ]
}


POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "ivanl002", "alias" : "alias1" } }
    ]
}

GET alias1/person/1


GET /ivanl002/_alias


DELETE /logs_20162801/_alias/current_day




PUT /logs_20162801
{
    "mappings" : {
        "type" : {
            "properties" : {
                "year" : {"type" : "integer"}
            }
        }
    },
    "aliases" : {
        "current_day" : {},
        "2016" : {
            "filter" : {
                "term" : {"year" : 2016 }
            }
        }
    }
}





```

