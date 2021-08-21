---
title: 如何实现延迟队列
date: 2021-08-21 11:10:43
tags: 
- design
categories: design
---

如何实现一个延迟队列？

#### (1) 数据库轮询
例如对于订单支付失效要求比较高的，每2S扫表一次检查过期的订单进行主动关单操作
优点是简单   
缺点是每分钟全局扫表，浪费资源，如果遇到表数据订单量即将过期的订单量很大，会造成关单延迟

#### (2) JDK的延迟队列
一个无界阻塞队列，该队列只有在延迟期满的时候才能从中获取元素，放入DelayQueue中的对象，是必须实现Delayed接口的
优点:效率高,任务触发时间延迟低。
缺点:(1)服务器重启后，数据全部消失，怕宕机 (2)集群扩展相当麻烦 (3)因为内存条件限制的原因，比如下单未付款的订单数太多，那么很容易就出现OOM异常 (4)代码复杂度较高

#### (3) 时间轮算法

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/redis/%E6%97%B6%E9%97%B4%E8%BD%AE.png)

时间轮算法可以类比于时钟，如上图箭头（指针）按某一个方向按固定频率轮动，每一次跳动称为一个 tick。这样可以看出定时轮由个3个重要的属性参数，ticksPerWheel（一轮的tick数），tickDuration（一个tick的持续时间）以及 timeUnit（时间单位），例如当ticksPerWheel=60，tickDuration=1，timeUnit=秒，这就和现实中的始终的秒针走动完全类似了。
如果当前指针指在1上面，我有一个任务需要4秒以后执行，那么这个执行的线程回调或者消息将会被放在5上。那如果需要在20秒之后执行怎么办，由于这个环形结构槽数只到8，如果要20秒，指针需要多转2圈。位置是在2圈之后的5上面（20 % 8 + 1）

优点:效率高,任务触发时间延迟时间比delayQueue低，代码复杂度比delayQueue低。
缺点:
- 服务器重启后，数据全部消失，怕宕机
- 集群扩展相当麻烦
- 因为内存条件限制的原因，比如下单未付款的订单数太多，那么很容易就出现OOM异常

#### (4) redis zset实现
利用redis的zset,zset是一个有序集合，每一个元素(member)都关联了一个score,通过score排序来取集合中的值。
zset常用命令
添加元素: ZADD key score member [[score member] [score member] ...]
按顺序查询元素: ZRANGE key start stop [WITHSCORES]
查询元素score: ZSCORE key member
移除元素: ZREM key member [member ...]

我们将订单超时时间戳与订单号分别设置为score和member,系统扫描第一个元素判断是否超时（zrange key 0 0 [withscores] ）

存在一个致命的硬伤，在高并发条件下，多消费者会取到同一个订单号.
解决方案: 
(1) 用分布式锁，但是用分布式锁，性能下降了，该方案不细说。
(2) 对ZREM的返回值进行判断，只有大于0的时候 (删除成功)，才可消费数据

优点:
(1) 由于使用Redis作为消息通道，消息都存储在Redis中。如果发送程序或者任务处理程序挂了，重启之后，还有重新处理数据的可能性。
(2) 做集群扩展相当方便
(3) 时间准确度高
缺点:
(1) 需要额外进行redis维护

#### (5)使用消息队列
我们可以采用rabbitMQ的延时队列。RabbitMQ具有以下两个特性，可以实现延迟队列

RabbitMQ可以针对Queue和Message设置 x-message-tt，来控制消息的生存时间，如果超时，则消息变为dead letter
lRabbitMQ的Queue可以配置x-dead-letter-exchange 和x-dead-letter-routing-key（可选）两个参数，用来控制队列内出现了deadletter，则按照这两个参数重新路由。

优缺点
优点: 高效,可以利用rabbitmq的分布式特性轻易的进行横向扩展,消息支持持久化增加了可靠性。缺点：本身的易用度要依赖于rabbitMq的运维.因为要引用rabbitMq,所以复杂度和成本变高