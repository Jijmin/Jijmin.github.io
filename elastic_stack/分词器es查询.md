GET _search
{
  "query": {
    "match_all": {}
  }
}

# 创建索引
PUT /test_index
# 查看现有索引
GET _cat/indices
# 删除索引
DELETE /test_index

# 指定 id 创建文档
PUT /test_index/doc/1
{
  "username": "alfred",
  "age": 1
}
# 不指定 id 创建文档
POST /test_index/doc
{
  "username": "tom",
  "age": 20
}
# 查询文档
GET /test_index/doc/_search
# 查询所有文档
GET /test_index/doc/_search
{
  "query": {
    "term": {
      "_id": "1"
    }
  }
}
# 批量创建文档
POST _bulk
{"index":{"_index":"test_index","_type":"doc","_id":"3"}}
{"username":"alfred","age":10}
{"delete":{"_index":"test_index","_type":"doc","_id":"1"}}
{"update":{"_id":"2","_index":"test_index","_type":"doc"}}
{"doc":{"age":"20"}}
# 批量查询文档
GET /_mget
{
  "docs": [
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "1"
    },
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "3"
    }
  ]
}

# 可以直接执行 analyzer 进行测试
POST _analyze
{
  "analyzer": "standard",
  "text": "hello world!"
}
# 可以直接指定索引中的字段进行测试
POST test_index/_analyze
{
  "field": "username",
  "text": "hello world"
}
# 可以自定义分词器进行测试
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase"],
  "text": "Hello World!"
}
# Standard 分词器
POST _analyze
{
  "analyzer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
# Simple 分词器
POST _analyze
{
  "analyzer": "simple",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
# Whitespace 分词器
POST _analyze
{
  "analyzer": "whitespace",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
# Stop 分词器
POST _analyze
{
  "analyzer": "stop",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
# Keyword 分词器
POST _analyze
{
  "analyzer": "keyword",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
# Pattern 分词器
POST _analyze
{
  "analyzer": "pattern",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
# 自定义分词-Character Filters
POST _analyze
{
  "tokenizer": "keyword",
  "char_filter": ["html_strip"],
  "text": "<p>I&apos;m so<b>happy</b>!</p>"
}
# 自定义分词-Tokenizer
POST _analyze
  {
    "tokenizer": "path_hierarchy",
    "text": "/one/two/three"
  } 
# 自定义分词-Token Filters
POST _analyze
{
  "text": "a Hello, world!",
  "tokenizer": "standard",
  "filter": [
    "stop",
    "lowercase",
    {
      "type": "ngram",
      "min_gram": 4,
      "max_gram": 4
    }
  ]
}
DELETE test_index
# 自定义分词器
PUT test_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "char_filter": [
            "html_strip"
          ],
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  }
}
POST test_index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Is this <b>a box</b>"
}
# 自定义分词2
PUT test_index2
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "emoticons"
          ],
          "tokenizer": "punctuation",
          "filter": [
            "lowercase",
            "english_stop"
          ]
        }
      },
      "tokenizer": {
        "punctuation": {
          "type": "pattern",
          "pattern": "[.,!?]"
        }
      },
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [
            ":) => _happy_",
            ":( => _sad_"
          ]
        }
      },
      "filter": {
        "english_stop": {
          "type": "stop",
          "stopwords": "_english_"
        }
      }
    }
  }
}
POST test_index2/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "I'm :) person,and you?"
}
# 索引时分词
PUT test_index3
{
  "mappings": {
    "doc": {
      "properties": {
        "title": {
          "type": "text",
          "analyzer": "whitespace"
        }
      }
    }
  }
}
# 查询的时候通过 analyzer 指定分词器
POST test_index3/_search
{
  "query": {
    "match": {
      "message": {
        "query": "hello",
        "analyzer": "standard"
      }
    }
  }
}
# 通过 index mapping 设置 search_analyzer 实现
PUT test_index4
{
  "mappings": {
    "doc": {
      "properties": {
        "title": {
          "type": "text",
          "analyzer": "whitespace",
          "search_analyzer": "standard"
        }
      }
    }
  }
}