## 消息队列面试场景

**Interviewer**：你好。

**candidate**：你好。

（面试官在你的简历上面看到了，呦，有个亮点，你在项目里用过 `MQ` ，比如说你用过 `ActiveMQ` ）

**Interviewer**：你在系统里用过消息队列吗？（面试官在随和的语气中展开了面试）

**candidate**：用过的（此时感觉没啥）

**Interviewer**：那你说一下你们在项目里是怎么用消息队列的？

**candidate**：巴拉巴拉，“我们啥啥系统发送个啥啥消息到队列，别的系统来消费啥啥的。比如我们有个订单系统，订单系统每次下一个新的订单的时候，就会发送一条消息到 `ActiveMQ` 里面去，后台有个库存系统负责获取消息然后更新库存。”

（部分同学在这里会进入一个误区，就是你仅仅就是知道以及回答你们是怎么用这个消息队列的，用这个消息队列来干了个什么事情？）

**Interviewer**：那你们为什么使用消息队列啊？你的订单系统不发送消息到 `MQ` ，直接订单系统调用库存系统一个接口，咔嚓一下，直接就调用成功，库存不也就更新了。

**candidate**：额。。。（楞了一下，为什么？我没怎么仔细想过啊，老大让用就用了），硬着头皮胡言乱语了几句。

（面试官此时听你楞了一下，然后听你胡言乱语了几句，开始心里觉得有点儿那什么了，怀疑你之前就压根儿没思考过这问题）

**Interviewer**：那你说说用消息队列都有什么优点和缺点？

（面试官此时心里想的是，你的 `MQ` 在项目里为啥要用，你没怎么考虑过，那我稍微简单点儿，我问问你消息队列你之前有没有考虑过如果用的话，优点和缺点分别是啥？）

**candidate**：This one。。。（确实平时没怎么考虑过这个问题啊。。。胡言乱语了）

（面试官此时心里已经更觉得你这哥儿们不行，平时都没什么思考）

**Interviewer**： `Kafka` 、 `ActiveMQ` 、 `RabbitMQ` 、 `RocketMQ` 都有什么区别？

（面试官问你这个问题，就是说，绕过比较虚的话题，直接看看你对各种 `MQ` 中间件是否了解，是否做过功课，是否做过调研）

**candidate**：我们就用过 `ActiveMQ` ，所以别的没用过。。。区别，也不太清楚。。。

（面试官此时更是觉得你这哥儿们平时就是瞎用，根本就没什么思考，觉得不行）

**Interviewer**：那你们是如何保证消息队列的高可用啊？

**candidate**：This one。。。我平时就是简单走 API 调用一下，不太清楚消息队列怎么部署的。。。

**Interviewer**：如何保证消息不被重复消费啊？如何保证消费的时候是幂等的啊？

**candidate**：啥？（ `MQ` 不就是写入&消费就可以了，哪来这么多问题）

**Interviewer**：如何保证消息的可靠性传输啊？要是消息丢失了怎么办啊？

**candidate**：我们没怎么丢过消息啊。。。

**Interviewer**：那如何保证消息的顺序性？

**candidate**：顺序性？什么意思？我为什么要保证消息的顺序性？它不是本来就有顺序吗？

**Interviewer**：如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时，说说怎么解决？

**candidate**：不是，我这平时没遇到过这些问题啊，就是简单用用，知道 `MQ` 的一些功能。

**Interviewer**：如果让你写一个消息队列，该如何进行架构设计啊？说一下你的思路。

**candidate**：。。。。。我还是走吧。。。。

---

这其实是面试官的一种面试风格，就是说面试官的问题不是发散的，而是从一个小点慢慢铺开。比如说面试官可能会跟你聊聊高并发话题，就这个话题里面跟你聊聊缓存、 `MQ` 等等东西，**由浅入深，一步步深挖**。

其实上面是一个非常典型的关于消息队列的技术考察过程，好的面试官一定是从你做过的某一个点切入，然后层层展开深入考察，一个接一个问，直到把这个技术点刨根问底，问到最底层。
