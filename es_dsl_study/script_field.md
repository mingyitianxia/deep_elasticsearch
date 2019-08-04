# 问题：能否在一个查询中 查询两个条件 在对两个结果进行除法计算

请教各位一个问题，我们有一个场景，想通过1个查询语句，计算两个查询结果的除法，比如，我有一个查询条件，用 idc: "BJ" 能统计出有100条数据符合要求 ，第二个条件 idc: "SH"，能统计出有200个数据，我现在想要取到 100 / 200 这个值 50% 这个数据，请问能有办法实现吗？请各位大神指点一下。

# 参考 es6.6版本

```
PUT test01_index/_doc/5
{
  "x_value":15,
  "y_value":3
}


POST test01_index/_doc/_search
{
  "script_fields": {
    "my_divide_field": {
      "script": {
        "lang": "expression",
        "source": "doc['y_value'].value != 0 ? doc['x_value'].value / doc['y_value'].value : 0"
      }
    }
  }
}
```
