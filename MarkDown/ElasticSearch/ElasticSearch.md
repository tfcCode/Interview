# 一、基本概念

索引（index）相当于 MySQL 的数据库

类型（type）相当于 Table（表）

文档（Document）相当于数据库的一条数据，ES 中的数据是 JSON 格式

> 将某条数据插入某个数据库某张表，就是将数据插入某个**索引**的某个**类型**中



# 二、查看基本信息

Get 请求：

* /_cat/nodes：查看所有节点
* /_cat/health：查看 ES 健康状况
* /_cat/master：查看主节点
* /_cat/indices：查看所有索引



# 三、基本操作

## 索引一个文档

### PUT 请求：必须携带 ID

也就是向某个索引下的某个类型插入一条数据

> 例：在 customer 索引下的 external 类型下保存 1 号（相当于唯一标识）数据

对同一数据发送多次，相当于更新操作，主要是更新版本号

```json
PUT: http://47.100.136.186:9200/customer/external/1
{
    "name": "徐凤年"
}
```

返回数据如下：

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```



### POST 请求：不一定携带 ID

使用 POST 不指定 ID 的话会自动生成一个唯一 ID，发送多次相同请求，每次 ID 都不一样

指定 ID 的话和 PUT 方式效果一样

```json
POST: http://47.100.136.186:9200/customer/external
{
    "name": "徐凤年"
}
```

返回数据如下：

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "5LmtKn4BzgIM7zmWUk_k",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```





## 查询文档(数据)

查询使用 `GET` 请求

> 例：查询 customer 索引中 external 类型下 1 号数据

```json
GET: http://47.100.136.186:9200/customer/external/1
```

返回结果如下

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 2,
    "_seq_no": 1,
    "_primary_term": 1,
    "found": true,
    "_source": {        //数据所在处
        "name": "徐凤年"
    }
}
```



## 更新文档

方式一：发送 POST 请求，携带`_update` 

> 例：更新 customer 索引中 external 类型下 1 号数据

**特点：发送多次同一个更新请求，版本号不会更新** 

```json
POST: http://47.100.136.186:9200/customer/external/1/_update
// 更新方式必须如以下格式
{
    "doc": {
        "name": "曹长卿",
        "age": 20
    }
}
```

方式二：使用 POST 请求不带 `_update` 或者使用 PUT 请求

**特点：发送多次同一个更新请求，版本号会更新**，且不用加 `doc` 属性

```json
POST: http://47.100.136.186:9200/customer/external/1
PUT: http://47.100.136.186:9200/customer/external/1
{
    "name": "曹长卿",
    "age": 20
}
```



## 删除文档

> 发送 DELETE 请求，例如删除 customer 索引中 external 类型下 1 号数据

```json
DELETE: http://47.100.136.186:9200/customer/external/1
```

也可以删除索引，**但没有提供删除类型** 



## 批量保存

* 必须是 `POST` 请求，且要携带 `_bulk` 
* 语法格式如下

```json
{"action":{metedata}}
{RequestBody}
```

`action`：表示动作，index 是新增，delete 是删除，update 是更新

`RequestBody`：数据

```json
POST: http://47.100.136.186:9200/customer/external/_bulk
{"index": {"_id": "1"}}  // 两行是一个数据
{"name": "姜泥"}
```

在 website 索引中的 blog 类型下创建数据（自动生成 id）

```json
{"index": {"_index":"website", "_type":"blog"}}
{"title":"index"}
```

在 website 索引中的 blog 类型下创建 123 号数据

```json
{"create": {"_index":"website", "_type":"blog", "_id":"123"}}
{"title":"create"}
```

删除 website 索引中 blog 类型下 123 号数据

```json
{"delete": {"_index": "website", "_type":"blog", "_id":"123"}}
```

更新 website 索引中 blog 类型下 123 号数据

```json
{"update":{"_index":"website", "_type":"blog", "_id":"123"}}
{"doc":{"title":"My update blog post"}}
```



# 四、条件检索：GET

需要携带`_search` 

方式一：直接在 url 后面拼接查询条件

```json
http://47.100.136.186:9200/bank/_search?q=*&sort=account_number:asc
```

方式二：将查询条件放在请求体中（**推荐使用这种方式**）

```json
GET /bank/_search  // 在 bank 索引下查询
{
  "query": { "match_all": {} },    // 查询条件
  "sort": [                        // 排序方式
    { "account_number": "asc" },    // 按账号降序
    { "balance": "desc"}
  ]
}
```

## 基本查询

```json
GET /bank/_search
{
  "query": { // 查询条件
    "match_all": {}           // 匹配所有
  },
  "sort": [  // 排序条件
    {
      "account_number": "asc"
    },
    {
      "balance": "desc"
    }
  ],
  "from": 0,   // 从第 0 条记录开始的 5 条记录
  "size": 5,
  "_source": ["balance","firstname","lastname"]   // 返回指定字段
}
```

## 精确、模糊查询：match、match_phrase

`match`：如果字段类型是非字符串，就是精确匹配，如果字段类型是字符串类型，那就是模糊匹配

```json
GET /bank/_search
{
  "query": {
    "match": {
      "balance": "16623"
    }
  }
}

{
  "query": {
    "match": {
      "address": "beijing wuhan"  // 包含 beijing、wuhan 的都能被查出来
    }
  }
}
```

`match_phrase`：模糊匹配

```json
GET /bank/_search
{
  "query": {
    "match_phrase": {
      "address": "mill lane" // 只能查出包含 "mill lane" 的元素
    }
  }
}
```



## 多字段匹配：multi_match

```json
GET /bank/_search
{
  "query": {
    "multi_match": {
      "query": "mill",
      "fields": ["address", "city"]
    }
  }
}
```



## 复合查询：bool

```json
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [ //必须匹配
        {
          "match": {
            "gender": "M"
          }
        },
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "must_not": [  // 必须不匹配
        {
          "match": {
            "age": "38"
          }
        }
      ],
      "should": [   // 应该：最好有，没有也可以
        {
          "match": {
            "lastname": "aaa"
          }
        }
      ]
    }
  }
}
```



## 结果过滤：filter

使用 filter 不会计算相关性得分

```json
GET /bank/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "gender": "M"
        }
      },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```



# 五、聚合

相当于 SQL 中 GROUP BY 和 聚合函数

基本格式

```json
"aggregations" : {
    "<aggregation_name>" : {
        "<aggregation_type>" : {
            <aggregation_body>
        }
        [,"meta" : {  [<meta_data_body>] } ]?
        [,"aggregations" : { [<sub_aggregation>]+ } ]?
    }
    [,"<aggregation_name_2>" : { ... } ]*
}
```



> 案例：搜索 address 中包含 mill 的所有人的平均年龄，但不显示这些人的详细信息

```json
GET /bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10        // 如果有多种可能，只取出前 10 中可能
      }
    },
    "ageAvg":{
      "avg": {
        "field": "age"
      }
    },
    "balanceAgg":{
      "avg": {
        "field": "balance"
      }
    }
  },
  "size": 0  // 不显示详细信息
}
```



# 六、映射

为每个字段指定数据类型，可以在创建索引的时指定字段类型，若创建索引时没有指定字段类型，ES 会自动推断

> 查询映射信息

```json
GET /my-index/_mapping
```

> 创建映射：发送 `PUT` 请求

```json
PUT /my-inde
{
  "mappings": {
    "properties": {
      "age": {
        "type": "integer"
      },
      "email": {
        "type": "keyword"
      },
      "name": {
        "type": "text"
      }
    }
  }
}
```

> 添加映射：发送`PUT`请求，需要携带`_mapping` 

```json
PUT /my-index/_mapping
{
  "properties": {
    "employee_id": {
      "type": "long",
      "index": false   // 该字段是否参与检索，默认都是为 true
    }
  }
}
```



## 修改映射

无法修改映射，因为一旦修改，ES 建立的检索信息都失效了

如果一定要修改映射，方法如下：

1、创建一个新的索引，配置新的映射信息

```json
PUT /newbank     / 创建新索引
{
  "mappings": {
    "properties": {
      "age": {
        "type": "integer"
      }
    }
  }
}
```

2、将旧索引的数据迁移到新索引：使用`_reindex` 

```json
POST _reindex
{
  "source": {
    "index": "bank",
    "type": "account"   // 如果索引中有类型，需要指定类型
  },
  "dest": {
    "index": "newbank"
  }
}
```



# 七、整合 Spring Boot

1、引入依赖：引入 high-level 版本的依赖（官方推荐）

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.4.2</version>
</dependency>
```

2、相关配置（参照官方文档）

```java
@Configuration
public class ElasticSearchConfig {
    @Bean
    public RestHighLevelClient esRestClient() {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("47.100.136.186", 9200, "http")));
        return client;
    }
}
```

3、配置好上述信息后就能使用`RestHighLevelClient`操作 ES 了，具体参照官方文档

```java
@Autowired
private RestHighLevelClient esClient;
```







