# 今日任务
1、搞清楚ES相关的概念

[Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)，这本书是了解ElasticSearch最好的书籍，相比较其他的文章，这本书能够原汁原味展示ElasticSearch的数据。


2、写一个关于分词的case


3、写一个查询分词的case


4、关于ES的性能


5、关于结巴分词和ES之间的关系

6、适合我自己业务的需求

[多词查询](https://www.elastic.co/guide/cn/elasticsearch/guide/current/match-multi-word.html)，我的项目中使用两个词进行查询，但是我的程序中出现的数据有可能会出现两次，因此在插入数据库的时候就会出现重复的问题，因此能够很大程度能够解决品质的问题。

一级数据和二级的数据，怎么说呢，我们用一级标签和二级标签的数据用一级的每一个词和二级的相关数据就可以进行匹配

[如何使用布尔匹配](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_how_match_uses_bool.html#_how_match_uses_bool)也就是说我们在这里可以使用
```java
{
  "bool": {
    "should": [
      { "term": { "title": "推送" }},
      { "match": { "title": "频繁 太多"   }}
    ]
  }
}
```


