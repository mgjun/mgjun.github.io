# Query DSL

* [Query and filter context](#Query-and-filter-context)
* [Compound queries](#Compound-queries)

## Query and filter context
#### Query context
Query context 查询的时候，会去计算相关性得分，并将相关性得分较高的放在前面返回。
> a query clause answers the question “How well 
> does this document match this query clause?” Besides deciding whether or not the document matches, the query clause also calculates a relevance score in the _score meta-field.

```
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ]
    }
  }
}
```
#### Filter context
Filter context 查询的时候，不会计算相关性得分，只是过滤数据是否满足查询条件,并且会将查询结果进行缓存。
> a query clause answers the question “Does this document match this query clause?” The answer is a simple Yes or No — no scores are calculated

filter context 比较适合结构数据的查询,如:
- 时间类型的数据是否在某个区间内
- 对数据的状态查询，status是否为published

```
GET /_search
{
  "query": { 
    "bool": { 
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```

## 最佳实践
在查询中如果需要分数匹配，则使用query context。其他情况尽量使用filter context
> 
Use query clauses in query context for conditions which should affect the score of matching documents (i.e. how well does the document match), and use all other query clauses in filter context.

## Compound queries

#### bool query
其他boolean query的聚集，使用Lucene BooleanQuery，并且包含几类查询类型,如must，should，must_not和filter。
- **must**和**should**遵循**Query context**，即会使用相关性得分，并根据相关性得分的高低来匹配查询结果。
- **must_not**和**filter**遵循**Filter context**，即不会使用相关性得分，直接匹配查询结果，并且查询之后会将查询结果进行缓存，方便下一次查询

```
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

#### boosting query
可以使用boosting query降级某些文档，而不将它们从搜索结果中排除。
- positive 文档必须匹配positive中的查询条件
- negative 如果某个文档匹配了positive，同时也匹配negative查询，那这个文档最后的相关性得分将会是(positive中的相关性得分*negative_boost)
  > negative_boost 取值范围为0到1

```
GET /_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "text" : "apple"
                }
            },
            "negative" : {
                 "term" : {
                     "text" : "pie tart fruit crumble tree"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```
#### constant score
用来包装filter query并且返回匹配指定相关性得分的文档
- filter 遵循filter query查询
- boost 可选参数，默认值为1
```
GET /_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}
```
