https://elasticsearch.cn/question/7948#!answer_12068

# 背景：
某公司在网上做一项调查，对公司员工的基本信息进行收集。目前得到一批数据进行分析。
现在有以下问题需要进行分析
数据：

100w家，每个公司有N个部门，每个部门有N个员工
 

#  要求：
查询满足【同一家公司内同一部门里，年龄在25-30岁之间的男性员工超过10人】的公司有哪些？
数据结构该如何设计？如图想了两种数据结构，不知道那种合适。
Elasticsearch能否像SQL的having count>N的这种方式作为query的条件筛选么？

ES版本5.4.3

#  备注：以上场景根据实际业务进行改编，数据量要比以上说明要多很多

以下建模使用了非nested存储，铺平存储。

```
DELETE company_infos
PUT company_infos
{
  "mappings": {
    "_doc": {
      "properties": {
        "no": {
          "type": "long"
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        },
        "sex": {
          "type": "keyword"
        },
        "age": {
          "type": "integer"
        },
        "company_name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        },
        "department": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        }
      }
    }
  }
}

POST /_bulk
{ "index" : { "_index" : "company_infos", "_type" : "_doc", "_id" : "1" } }
{ "no" : 1, "name":"zhangsan", "age":25, "company_name":"xiaomi", "department":"waa","sex":"man"}
{ "index" : { "_index" : "company_infos", "_type" : "_doc", "_id" : "2" } }
{ "no" : 2, "name":"lisi", "age":30, "company_name":"xiaomi", "department":"waa","sex":"man"}
{ "index" : { "_index" : "company_infos", "_type" : "_doc", "_id" : "3" } }
{ "no" : 3, "name":"wangwu", "age":27, "company_name":"huawei", "department":"haa","sex":"man"}
{ "index" : { "_index" : "company_infos", "_type" : "_doc", "_id" : "4" } }
{ "no" : 4, "name":"zhaoliu", "age":29, "company_name":"huawei", "department":"haa","sex":"man"}
{ "index" : { "_index" : "company_infos", "_type" : "_doc", "_id" : "5" } }
{ "no" : 5, "name":"fengqi", "age":35, "company_name":"huawei", "department":"haa","sex":"man"}
{ "index" : { "_index" : "company_infos", "_type" : "_doc", "_id" : "6" } }
{ "no" : 6, "name":"laoliu", "age":28, "company_name":"huawei", "department":"haa","sex":"man"}



POST company_infos/_search
{
  "size": 0,
  "aggs": {
    "company_names_aggs": {
      "terms": {
        "field": "company_name.keyword",
        "size": 10
      },
      "aggs": {
        "department_aggs": {
          "terms": {
            "field": "department.keyword"
          },
          "aggs": {
            "sex_aggs": {
              "terms": {
                "field": "sex",
                "size": 10
              },
              "aggs": {
                "employee_count": {
                  "range": {
                    "field": "age",
                    "ranges": [
                      {
                        "from": 25,
                        "to": 31
                      }
                    ]
                  },
                  "aggs": {
                    "my_filter": {
                      "bucket_selector": {
                        "buckets_path": {
                          "the_doc_count": "_count"
                        },
                        "script": "params.the_doc_count > 1"
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```
