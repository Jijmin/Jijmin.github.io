DELETE my_index
DELETE test_index

GET _search
{
  "query": {
    "match_all": {}
  }
}

# 自定义 Mapping dynamic
PUT my_index
{
  "mappings": {
    "doc": {
      "dynamic": "strict",
      "properties": {
        "title": {
          "type": "text"
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        }
      }
    }
  }
}
PUT my_index/doc/2
{
  "title": "dynamic false",
  "name": "zy",
  "age": 24,
  "sex": 0
}
GET my_index/doc/_search
{
  "query": {
    "match": {
      "sex":0
    }
  }
}
# 自定义 Mapping dynamic
PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "first_name": {
          "type": "text",
          "copy_to": "full_name"
        },
        "last_name": {
          "type": "text",
          "copy_to": "full_name"
        },
        "full_name": {
          "type": "text"
        }
      }
    }
  }
}
PUT my_index/doc/1
{
  "first_name": "John",
  "last_name": "Smith"
}
GET my_index/_search
{
  "query": {
    "match": {
      "full_name": {
        "query": "John Smith",
        "operator": "and"
      }
    }
  }
}

# 自定义 Mapping index
PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie": {
          "type": "text",
          "index": false
        }
      }
    }
  }
}
PUT my_index/doc/1
{
  "cookie": "name=alfred"
}
GET my_index/_search
{
  "query": {
    "match": {
      "cookie": "name"
    }
  }
}

# 自定义 Mapping index_options
PUT my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "cookie": {
          "type": "text",
          "index_options": "offsets"
        }
      }
    }
  }
}
GET my_index/_mapping

# 自定义 Mapping null_value
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "status_code": {
          "type": "keyword",
          "null_value": "NULL"
        }
      }
    }
  }
}
# es 可以自动识别文档字段类型
PUT /test_index/doc/1
{
  "username": "alfred",
  "age": 1
}
GET /test_index/_mapping
GET /my_index/_mapping
# 定义日期识别格式
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_date_formats": [
        "MM/dd/yyyy"
      ]
    }
  }
}
PUT my_index/my_type/1
{
  "create_date": "09/25/2019"
}
# 关闭日期自动识别机制
PUT my_index
{
  "mappings": {
    "my_type": {
      "date_detection": false
    }
  }
}
# numeric_detection 可以开启字符串中数字的自动识别
PUT my_index
{
  "mappings": {
    "my_type": {
      "numeric_detection": true
    }
  }
}
PUT my_index/my_type/1
{
  "my_float": "1.0",
  "my_integer": 1,
  "message-a": "<p>aaa</p>"
}
GET my_index/_mapping
# 所有字符串类型都设定为 keyword 类型，即默认不分词
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
        {
          "strings": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "keyword"
            }
          }
        }
      ]
    }
  }
}
# 以 message 开头的字段都设置为 text 类型
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
        {
          "message_as_text": {
            "match_mapping_type": "string",
            "match": "message*",
            "mapping": {
              "type": "text"
            }
          }
        }
      ]
    }
  }
}
# double 类型设定为 float
PUT test_index
{
  "mappings": {
    "doc": {
      "dynamic_templates": [
        {
          "double_as_float": {
            "match_mapping_type": "double",
            "mapping": {
              "type": "float"
            }
          }
        }
      ]
    }
  }
}
# 索引模板，主要用于在新建索引时自动应用预先设定的配置
PUT _template/test_template
{
  "index_patterns": [
    "te*",
    "bar*"
  ],
  "order": 0,
  "settings": { 
    "number_of_shards": 1
  },
  "mappings": {
    "doc": {
      "_source": {
        "enabled": false
      },
      "properties": {
        "name": {
          "type": "keyword"
        }
      }
    }
  }
}
PUT _template/test_template2
{
  "index_patterns": [
    "test*"
  ],
  "order": 1,
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "doc": {
      "_source": {
        "enabled": true
      }
    }
  }
}
# 获取与删除的 API
GET _template
GET _template/test_template
DELETE _template/test_template2