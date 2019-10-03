### Mapping
1. 类似数据库中的表结构定义，主要作用如下
2. 定义 Index 下的字段名(Field Name)
3. 定义字段的类型，比如数值型、字符串型、布尔型等
4. 定义倒排索引相关的配置，比如是否索引、记录 position 等
5. Mapping 中的字段类型一旦设定后，禁止直接修改，原因如下：
  - Lucene 实现的带牌索引生成后不允许修改
6. 重新建立新的索引，然后做 reindex 操作
7. 允许新增字段
8. 通过 dynamic 参数来控制字段的新增
  - true(默认)允许自动新增字段
  - false 不允许自动新增字段，但是文档可以正常写入，但无法对字段进行查询等操作
  - strict 文档不能写入，报错
```
GET /test_index/_mapping
```

### 自定义 Mapping 的 api
1. dynamic
```
PUT my_index
{
  "mappings": {  # mappings 关键字
    "doc": { # type 名称
      "dynamic": true,
      "properties": { # 字段名称和类型定义
        "title": {
          "type": "text" # 字段类型
        },
        "name": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        },
        "social_networks": {
          "dynamic": true, # 也可以在这里配置
          "properties": {}
        }
      }
    }
  }
}
```
2. copy_to
  - 将该字段的值复制到目标字段，实现类似_all的作用
  - 不会出现在_source中，只用来搜索
  ```
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
  ```
3. index
  - 控制当前字段是否索引，默认为true，即记录索引，false不记录，即不可搜索
  ```
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
  ```
4. index_options
  - 用于控制倒排索引记录的内容，有如下4中配置
    - docs 只记录 doc id
    - freqs 记录 doc id 和 term frequencies
    - positions 记录 doc id、term frequencies 和 term position
    - offsets 记录 doc id、term frequencies、term position 和 character offsets
  - text 类型默认配置为 positions，其他默认为 docs
  - 记录内容越多，占用空间越大
  ```
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
  ```
5. null_value
  - 当字段遇到 null 值时的处理策略，默认为 null，即空值，此时 es 会忽略该值。可以通过设定该值设定字段的默认值
  ```
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
  ```

### 数据类型
1. 核心数据类型
  - 字符串型：text(分词)、keyword(不分词)
  - 数值型：long、integer、short、byte、double、float、half_float、scaled_float
  - 日期类型：date
  - 布尔类型：boolean
  - 二进制类型：binary
  - 范围类型：integer_range、float_range、long_range、double_range、date_ranges
2. 复杂数据类型
  - 数组类型：array
  - 对象类型：object
  - 嵌套类型：nested object
3. 地理位置数据类型
  - geo_point
  - geo_shape
4.  专用类型
  - 记录 ip 地址 ip
  - 实现自动补全 completion
  - 记录分词数 token_count
  - 记录字符串 hash 值 murmur3
  - percolator
  - join
5. 多字段特性 multi-fields
  - 允许对同一个字段采用不同的配置，比如分词，常见例子如对人名实现拼音搜索，只需要在人名中新增一个子字段为 pinyin 即可
  - 通过参数 fields 设置
  ```
  # response
  {
    "text_index": {
      "mappings": {
        "doc": {
          "properties": {
            "username": {
              "type": "text",
              "fields": {
                "pinyin": {
                  "type": "text",
                  "analyzer": "pinyin"
                }
              }
            }
          }
        }
      }
    }
  }
  GET test_index/_search
  {
    "query": {
      "match": {
        "username.pinyin": "hanhan"
      }
    }
  }
  ```

### Dynamic Mapping
1. es 可以自动识别文档字段类型，从而降低用户使用成本，如下所示：
```
PUT /test_index/doc/1
{
  "username": "alfred",
  "age": 1
}
GET /test_index/_mapping

# es 自动识别 age 为 long 类型，username 为 text 类型
```
2. es 是依靠 JSON 文档的字段类型类实现自动识别字段类型，支持的类型如下：

JSON类型 | ES 类型
------------- | -------------
null | 忽略
boolean | boolean
浮点类型 | float
整数 | long
object | object
array | 由第一个非 null 值的类型决定
string | 匹配为日期则设为 date 类型(默认开启)； 匹配为数组的话设为 float 或 long 类型(默认关闭)；设为 text 类型，并附带 keyword 的子字段
3. 日期的自动识别可以自行配置日期格式，以满足各种需求
  - 默认是 `["strict_date_optional_time","yyyy/MM/dd HH:mm:ss Z||yyyy/MM/dd Z"]`
  - strict_date_optional_time 是 ISO datetime 的格式，完整格式类似下面：
    - YYYY-MM-DDThh:mm:ssTZD(eg 1997-07-16T19:20:30+01:00)
  - dynamic_date_formats 可以自定义日期类型
  - date_detection 可以关闭日期自动识别的机制
  ```
  # 自定义日期识别格式
  PUT my_index
  {
    "mappings": {
      "my_type": {
        "dynamic_date_formats": ["MM/dd/yyyy"]
      }
    }
  }
  PUT my_index/my_type/1
  {
    "create_date": "09/25/2019"
  }

  # 关闭日期自动识别机制
  PUT my_index{
    "mappings": {
      "my_type": {
        "date_detection": false
      }
    }
  }
  ```
4. 字符串是数字时，默认不会自动识别为整型，因为字符串中出现数字是完全合理的
  - numeric_detection 可以开启字符串中数字的自动识别，如下所示：
  ```
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
    "my_integer": 1
  }
  GET my_index/_mapping
  ```

### Dynamic Templates
1. 允许根据 es 自动识别的数据类型、字段名等来动态设定字段类型，可以实现如下效果：
  - 所有字符串类型都设定为 keyword 类型，即默认不分词
  - 所有以 message 开头的字段都设定为 text 类型，即分词
  - 所有以 long_开头的字段都设定为 long 类型
  - 所有自动匹配为 double 类型的都设定为 float 类型，以节省空间
  ```
  PUT test_index
  {
    "mappings": {
      "doc": {
        "dynamic_templates": [ # 数组，可指定多个匹配规则
          {
            "strings": { # template 的名称
              "match_mapping_type": "string", # 匹配规则
              "mapping": { # 设置 mapping 信息
                "type": "keyword"
              }
            }
          }
        ]
      }
    }
  }
  ```
2. 匹配规则一般有如下几个参数
  - match_mapping_type 匹配 es 自动识别的字段类型，如 boolean，long，string 等
  - match，unmatch 匹配字段名
  - path_match，path_unmatch 匹配路径
  ```
  PUT test_index
  {
    "mappings": {
      "doc": {
        "dynamic_templates": [
          {
            "strings_as_keywords": [
              {
                "match_mapping_type": "string",
                "mapping": {
                  "type": "keyword"
                }
              }
            ]
          }
        ]
      }
    }
  }
  ```
3. 以 message 开头的字段都设置为 text 类型
```
PUT test_index
{
  "mappings": {
    "doc": {
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
```
4. double 类型设定为 float，节省空间
```
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
```

### 自定义 Mapping 的操作步骤如下
1. 写入一条文档到 es 的临时索引中，获取 es 自动生成的 mapping
2. 修改步骤1得到的 mapping，自定义相关配置
3. 使用步骤2的 mapping 创建实际所需索引

### 索引模板
1. 索引模板，英文为 Index Template，主要用于在新建索引时自动应用预先设定的配置，简化索引创建的操作步骤
  - 可以设定索引的配置和 mapping
  - 可以有多个模板，根据 order 设置，order 大的覆盖小的配置
  - 索引模板 API，endpoint 为 _template，如下所示
  ```
  PUT _template/test_template # template 名称
  {
    "index_patterns": ["te*", "bar*"], # 匹配的索引的名称
    "order": 0, # order 顺序配置
    "settings": { # 索引的配置
      "number_of_shards": 1
    },
    "mappings": {
      "doc": {
        "_source": {
          "enabled": false # 匹配的索引默认是不会记录原始数据的
        },
        "properties": {
          "name": {
            "type": "keyword"
          }
        }
      }
    }
  }
  # 创建索引并查看配置
  PUT test_index
  GET test_index
  ```
2. 获取与删除的 API 如下
```
GET _template
GET _template/test_template
DELETE _template/test_template
```