# 对所有的索引进行查询 
GET /_search
# 对指定索引进行查询
GET /my_index/_search
# 可以指定多个
GET /my_index,my_index1/_search
# 使用通配符指定索引
GET /my_*/_search

# 写入测试数据
PUT test_search_index
{
  "settings": {
    "index": {
      "number_of_shards": 1
    }
  }
}
POST test_search_index/doc/_bulk
{"index":{"_id":"1"}}
{"username":"alfred way","job":"java engineer","age":18,"birth":"1990-01-02","isMarried":false}
{"index":{"_id":"2"}}
{"username":"alfred","job":"java senior engineer and java specialist","age":28,"birth":"1980-05-07","isMarried":true}
{"index":{"_id":"3"}}
{"username":"lee","job":"java and ruby engineer","age":22,"birth":"1985-08-07","isMarried":false}
{"index":{"_id":"4"}}
{"username":"alfred hunior way","job":"ruby engineer","age":23,"birth":"1989-08-07","isMarried":false}

# search API
GET test_search_index/_search?q=alfred
# 查看查询细则
GET test_search_index/_search?q=alfred
{
  "profile": true
}
GET test_search_index/_search?q=username:alfred
GET test_search_index/_search?q=username:alfred way
GET test_search_index/_search?q=username:"alfred way"
GET test_search_index/_search?q=username:(alfred way)
# 布尔操作符
GET test_search_index/_search?q=username:alfred AND way
GET test_search_index/_search?q=username:(alfred AND way)
GET test_search_index/_search?q=username:(alfred NOT way)
GET test_search_index/_search?q=username:(alfred +way)
GET test_search_index/_search?q=username:(alfred %2Bway)
# 范围查询，支持数值和日期
GET test_search_index/_search?q=username:alfred AND age:>20
GET test_search_index/_search?q=birth:(>1980 AND <1990)
# 通配符查询
GET test_search_index/_search?q=username:alf*
# 正则表达式匹配
GET test_search_index/_search?q=username:/[a]?l.*/
# 模糊匹配
GET test_search_index/_search?q=username:alfd~1
GET test_search_index/_search?q=username:alfd~2
GET test_search_index/_search?q=job:"java engineer"
GET test_search_index/_search?q=job:"java engineer"~1
GET test_search_index/_search?q=job:"java engineer"~2

# 对字段做全文检索
GET test_search_index/_search
{
  "profile": true,
  "query": {
    "match": {
      "username": "alfred way"
    }
  }
}
# 通过 operator 参数可以控制单词间的匹配关系，可选项为 or 和 and
GET test_search_index/_search
{
  "profile": true,
  "query": {
    "match": {
      "username": {
        "query": "alfred way",
        "operator": "and"
      }
    }
  }
}
# 通过 minimum_should_match 参数可以控制需要匹配的单词数
GET test_search_index/_search
{
  "query": {
    "match": {
      "username": {
        "query": "alfred way",
        "minimum_should_match": "2"
      }
    }
  }
}
# 通过 explain 参数来查看具体的计算方法，分片数为1
GET test_search_index/_search
{
  "explain": true,
  "query": {
    "match": {
      "username": "alfred way"
    }
  }
}
# 对字段作检索，有顺序要求
GET test_search_index/_search
{
  "query": {
    "match_phrase": {
      "job": "java engineer"
    }
  }
}
# 通过 slop 参数可以控制单词间的间隔
GET test_search_index/_search
{
  "query": {
    "match_phrase": {
      "job": {
        "query": "java engineer",
        "slop": 1
      }
    }
  }
}
# 指明默认查询的字段名
GET test_search_index/_search
{
  "query": {
    "query_string": {
      "default_field": "username",
      "query": "alfred AND way"
    }
  }
}
# 指明默认查询的字段名
GET test_search_index/_search
{
  "query": {
    "query_string": {
      "fields": ["username", "job"],
      "query": "alfred AND way"
    }
  }
}
# Simple Query String Query
GET test_search_index/_search
{
  "query": {
    "simple_query_string": {
      "query": "alfred +way",
      "fields": ["username"]
    }
  }
}
# 将查询语句作为整个单词进行查询，即不对查询语句做分词处理
GET test_search_index/_search
{
  "query": {
    "term": {
      "username": "alfred way"
    }
  }
}
# 范围查询主要针对数值和日期类型
GET test_search_index/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
# 针对日期做范围查询
GET test_search_index/_search
{
  "query": {
    "range": {
      "birth": {
        "gte": "1990-01-01",
        "lte": "now-20y"
      }
    }
  }
}
# 该查询将其内部的查询结果文档得分都设定为1或者 boost 的值,多用于结合 bool 查询实现自定义得分
GET test_search_index/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match": {
          "username": "alfred"
        }
      }
    }
  }
}
# Filter 查询只过滤符合条件的文档，不会进行相关性算分
GET test_search_index/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "username": "alfred"
          }
        }
      ]
    }
  }
}
# 查询 username 包含 alfred 并且 job 包含 specialist 关键词的文档列表
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "username": "alfred"
          }
        },
        {
          "match": {
            "job": "specialist"
          }
        }
      ]
    }
  }
}
# 查询 job 中包含 java 关键词但不包含 ruby 关键词的文档列表
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "job": "java"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "job": "ruby"
          }
        }
      ]
    }
  }
}
# 只包含 should 时，文档必须满足至少一个条件
GET test_search_index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "job": "java"
          }
        },
        {
          "term": {
            "job": "ruby"
          }
        },
        {
          "term": {
            "job": "specialist"
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
# 同时包含 should 和 must 时，文档不必满足 should 中的条件，但是如果满足条件，会增加相关性得分
GET test_search_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "username": "alfred"
          }
        }
      ],
      "should": [
        {
          "term": {
            "job": "ruby"
          }
        }
      ]
    }
  }
}
# 获取符合条件的文档数
GET test_search_index/_count
{
  "query": {
    "match": {
      "username": "alfred"
    }
  }
}
# 过滤返回结果中 _source 中的字段
GET test_search_index/_search?_source=username
# 不返回 _source
GET test_search_index/_search
{
  "_source": false 
}
# 返回部分字段
GET test_search_index/_search
{
  "_source": ["username", "age"]
}
GET test_search_index/_search
{
  "_source": {
    "includes": "*i*",
    "excludes": "birth"
  }
}