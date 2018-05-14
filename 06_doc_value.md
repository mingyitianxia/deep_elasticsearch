# Elasticsearch index_time 解读

## 1、fielddata官网原文
https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html

Most fields are indexed by default, which makes them searchable. Sorting, aggregations, and accessing field values in scripts, however, requires a different access pattern from search.

Search needs to answer the question "Which documents contain this term?", while sorting and aggregations need to answer a different question: "What is the value of this field for thisdocument?".

Most fields can use index-time, on-disk doc_values for this data access pattern, but text fields do not support doc_values.
Instead, text fields use a query-time in-memory data structure called fielddata. This data structure is built on demand the first time that a field is used for aggregations, sorting, or in a script. It is built by reading the entire inverted index for each segment from disk, inverting the term ↔︎ document relationship, and storing the result in memory, in the JVM heap.

正确的翻译参考：

索引时字段层权重提升（Index-Time Field-Level Boosting）

字段是可以被搜索的名值对，不同的事件可能拥有不同的字段。Splunk支持索引时（index time）和搜索时（search time）的字段抽取（fields extraction）

所以：index time 翻译为： 索引时。

## 2、doc_value 引申解读
参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.2/modules-fielddata.html

The field data cache is used mainly when sorting on or computing aggregations on a field. It loads all the field values to memory in order to provide fast document based access to those values. The field data cache can be expensive to build for a field, so its recommended to have enough memory to allocate it, and to keep it loaded.

The amount of memory used for the field data cache can be controlled using indices.fielddata.cache.size. Note: reloading the field data which does not fit into your cache will be expensive and perform poorly.

indices.fielddata.cache.size
The max size of the field data cache, eg 30% of node heap space, or an absolute value, eg 12GB. Defaults to unbounded. Also see Field data circuit breakeredit.

These are static settings which must be configured on every data node in the cluster.

引申解读2：
https://www.elastic.co/guide/en/elasticsearch/reference/6.2/doc-values.html

Most fields are indexed by default, which makes them searchable. The inverted index allows queries to look up the search term in unique sorted list of terms, and from that immediately have access to the list of documents that contain the term.

Sorting, aggregations, and access to field values in scripts requires a different data access pattern. Instead of looking up the term and finding documents, we need to be able to look up the document and find the terms that it has in a field.

Doc values are the on-disk data structure, built at document index time, which makes this data access pattern possible. They store the same values as the _source but in a column-oriented fashion that is way more efficient for sorting and aggregations. Doc values are supported on almost all field types, with the notable exception of analyzed string fields.


All fields which support doc values have them enabled by default. If you are sure that you don’t need to sort or aggregate on a field, or access the field value from a script, you can disable doc values in order to save disk space:
```
PUT my_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "status_code": { 
          "type":       "keyword"
        },
        "session_id": { 
          "type":       "keyword",
          "doc_values": false
        }
      }
    }
  }
}
```
The status_code field has doc_values enabled by default.
The session_id has doc_values disabled, but can still be queried.

## 小结：
1、doc_values是存储在磁盘的数据结构，在文档建立索引的时候创建。

2、对比：field_data缓存主要用于在字段上进行排序或计算聚合。 它将所有字段值加载到内存中，以便为这些值提供基于文档的快速访问。
