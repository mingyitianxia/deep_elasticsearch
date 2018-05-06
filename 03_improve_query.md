# 1、基础查询
```
GET library/_search
{
  "_source":{
    "includes":["title"]
  },
  "query": {
    "match": {
      "title":{
        "query": "Crime Punishment",
        "operator": "and"
      }
    }
  }
}
```
# 2、多字段检索
```
GET library/_search
{
  "_source":{
    "includes":["title"]
  },
  "query": {
    "multi_match": {
        "query": "Crime Punishment",
       "fields": [
         "title^100",
         "text^10"
         ]
      }
    }
  }
  ```
  
  # 3、短语检索
  ```
   GET library/_search
  {
  "_source": {
    "includes": [
      "title"
    ]
  },
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "Crime Punishment",
            "fields": [
              "title^100",
              "text^10"
            ]
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": "crime punishment"
          }
        },
        {
          "match_phrase": {
            "author": "crime punishment"
          }
        }
      ]
    }
  }
}
```

# 4、引入slop设置单词间的最大间隔，
最大间隔之内的单词可以被认为与查询中的短语匹配。
```
GET library/_search
{
  "_source": {
    "includes": [
      "title"
    ]
  },
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "Crime Punishment",
            "fields": [
              "title^100",
              "text^10"
            ]
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": {
              "query": "crime punishment",
              "slop": 1
            }
          }
        },
        {
          "match_phrase": {
            "author": {
              "query": "crime punishment",
              "slop": 1
            }
          }
        }
      ]
    }
  }
}
```

# 5、过滤到垃圾信息——must_not来帮忙
```
GET library/_search
{
  "_source": {
    "includes": [
      "title"
    ]
  },
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "Crime Punishment",
            "fields": [
              "title^100",
              "text^10"
            ]
          }
        }
      ],
      "should": [
        {
          "match_phrase": {
            "title": {
              "query": "crime punishment",
              "slop": 1
            }
          }
        },
        {
          "match_phrase": {
            "author": {
              "query": "crime punishment",
              "slop": 1
            }
          }
        }
      ],
      "filter": {
        "bool": {
          "must_not": [
            {
              "term": {
                "redirect": "true"
              }
            },
            {
              "term": {
                "special": "true"
              }
            }
          ]
        }
      }
    }
  }
}

```
# 6、引入boost提升权重
```
{
  "_source": {
    "includes": [
      "title"
    ]
  },
  "query": {
    "function_score": {
      "boost": 100,
      "query": {
        "match_phrase": {
          "title": {
            "query": "Crime Punishment",
            "slop": 1
          }
        }
      }
    }
  }
}
```

# 7、可纠正拼写错误——借助：minimum_should_match
```
GET library/_search
{
  "query": {
    "query_string": {
      "query": "Crime Punishment",
      "default_field": "title",
      "minimum_should_match": "80%"
    }
  }
}
```
