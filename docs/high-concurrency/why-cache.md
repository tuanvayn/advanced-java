## Interview questions

How the cache is used in the project？Why use caching？缓存使用不当会造成什么后果？

## Interviewer psychoanalysis

This question，互联网公司必问，要是一个人连缓存都不太清楚，那确实比较尴尬。

Just ask about caching，上来第一个问题，肯定是先问问你项目哪里用了缓存？为啥要用？不用行不行？如果用了以后可能会有什么不良的后果？

这就是看看你对缓存这个东西背后有没有思考，如果你就是傻乎乎的瞎用，没法给面试官一个合理的解答，那面试官对你印象肯定不太好，觉得你平时思考太少，就知道干活儿。

## Anatomy of an interview question

### How the cache is used in the project？

This one，Need to combine the business of your own project。

### Why use caching？

With caching，There are two main uses：**Performance**、**High concurrency**。

#### Performance

假设这么个场景，你有个操作，一个请求过来，吭哧吭哧你各种乱七八糟操作 mysql，半天查出来一个结果，耗时 600ms。但是这个结果可能接下来几个小时都不会变了，或者变了也可以不用立即反馈给用户。那么此时咋办？

缓存啊，折腾 600ms 查出来的结果，扔缓存里，一个 key 对应一个 value，下次再有人查，别走 mysql 折腾 600ms 了，直接从缓存里，通过一个 key 查出来一个 value，2ms 搞定。性能提升 300 倍。

就是说对于一些需要复杂操作耗时查出来的结果，且确定后面不怎么变化，但是有很多读请求，那么直接将查询出来的结果放在缓存中，后面直接读缓存就好。

#### High concurrency

mysql 这么重的数据库，压根儿设计不是让你玩儿高并发的，虽然也可以玩儿，但是天然支持不好。mysql 单机支撑到 `2000QPS` 也开始容易报警了。

所以要是你有个系统，高峰期一秒钟过来的请求有 1 万，那一个 mysql 单机绝对会死掉。你这个时候就只能上缓存，把很多数据放缓存，别放 mysql。缓存功能简单，说白了就是 `key-value` 式操作，单机支撑的并发量轻松一秒几万十几万，支撑高并发 so easy。单机承载并发量是 mysql 单机的几十倍。

> 缓存是走内存的，内存天然就支撑高并发。

### What are the consequences of using caching?？

常见的缓存问题有以下几个：

-   [The cache is inconsistent with the database doublewrite](/docs/high-concurrency/redis-consistence.md)
-   [Cache avalanche、Cache walkthrough、Cache breakdown](/docs/high-concurrency/redis-caching-avalanche-and-caching-penetration.md)
-   [Cache concurrency contention](/docs/high-concurrency/redis-cas.md)

点击超链接，可直接查看缓存相关问题及解决方案。
