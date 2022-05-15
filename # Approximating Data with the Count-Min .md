# Approximating Data with the Count-Min Data Structure

## 引言

我们引入Count Tracking问题，这个问题是membership问题的广义问题，也是现代数据处理场景的核心问题。在这个问题中有大量的item，其中每一个item都有一个随着时间变化的frequency。这里有一个在记录搜索词条的使用案例。

考虑在一个很热门的网站想要记录在这个网站搜索词条的统计。其中一个方法是统计搜索词条的日志，这种方式能够得到每个词条都锁量的精准统计。但是，日志很快就会变得很大。搜索词条统计问题是tracking 问题的一个案例。使用树结构或者hash表来统计相同的词条（query）出现的精准次数比较慢而且比较浪费资源（内存）。在这种应用场景，我们可以忍受一些不精准性。通常我们只对那些频繁被搜索的词条感兴趣。对于query中存在一些fuzzies也是能够接受的。因此，我们在精准性和方案的高效率之间做平衡。tradeoff是sketches问题的核心。

在线上零售商场景，item表示待售的商品，frequency就是每一个商品（item）的销量；在股票交易市场，item就是股票，frequency就是在一天内股票交易量；此外， count tracking可以被用在派生数据，比方在分布式网址活着在不同的时期数据的差异性。此外，在更通用的场景下，许多具有海量数据的任务也可以使用count tracking解决，如：数据统计、挖掘、分类、异常检测等等。
任何Count Tracking数据结构一定要提供两个方法
更新方法
```java
UPDATE(i,c) // update the freqency of item i by c
```
预估方法
```java
ESTIMATE(i) // 
```

