# Elasticsearch

## Elasticsearch 简介
Elasticsearch(es)是一个分布式文档存储服务。在es中文档保存为json格式，因此可以存储复杂结构的数据。es使用**倒排序索引**来保证查询速度，一般在1s内返回。
在默认情况下，es会为index下的所有字段建立索引，并且每个字段的索引都是单独的，优化过的文档数据结构(json).比如，text field会使用倒排索引，数字和地理位置数据会使用BKD trees。
同时，我们可以为一个field基于不同目的创建多个索引，比如，可以为一个string field创建一个倒排序索引用于全文检索，同时创建一个关键字用于排序或者聚合查询;或者在多语言查询中，可以使用多个语言分析器(使用分析器链的方式来实现)来处理用户的输入。


## 查找和分析数据
es REST api支持多种查询方式，如[Query DSL](./queryDSL.md), full test queries，[SQL-style queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-overview.html)

## Scalability and resilience: clusters, nodes, and shards
es的index存储在一个或多个shard中，每个shard部署在一个或多个node中，一组功能相近的nodes组成一个es的cluster。在运行过程中，es可以通过添加node的方式来实现扩容，添加node后，es会自动实现数据的分发以及查询。
es的shard分为主shard和备份shard，每个文档归属于一个主shard。备份shard是一个主shard的备份，用于查询或当主shard出现故障时来顶替主shard。**一个主shard可以有一个或多个备份shard。**
**当创建一个index之后，主shard数量固定不能改变，但备份shard可以随意改变。**
对于shard的大小，如果shard越大，es如果需要rebalance，那此时花费的时间就越长。
对于shard的数量，如果数量越多，查询和维护这些shard的成本也就越高。

> 一些最佳实践:
> - shard的大小应该保持在几个或几十GB范围内。对于时间序列数据，应该保持在20到40GB范围内。
> - 为了避免每个shard存放大量数据，每GB占用的堆空间应该小于20。

对于在cluster级别的高可用，es提供了一种Cross-cluster replication (CCR)。CCR会自动同步主cluster的indicies，并且当主cluster不可用时，备份cluster会自动切换为主cluster提供服务；同时备份cluster可以处理读请求来分摊主cluster的压力。


* [Query DSL](./queryDSL.md)