---
title: 库存服务提性能
date: 2021-08-01 11:10:43
tags: 
- design
categories: design
---

#### 提性能

##### 读优化
多级缓存：让数据靠近计算
链路优化：缩短耗时
周期库存⽅案：提⾼周期库存查询性能

##### 写优化
流量漏⽃：缓存预扣减少数据库的⽆效访问
分库分表：提升数据库的整体写⼊能⼒( 窄表设计下单分片物理机可承载4500，云服务器可承载3000，仅供参考，具体以业务测试为准 ) 
并⾏扣减：降低库存扣减的响应时间
分库存：解决单行更新能力不足问题，（一般单行更新的QPS在500以内）
合并请求：异步批量，将多次扣减累计计数，集中成一次扣减，从而实现了将串行处理变成了批处理。大大减轻更新压力。

当前峰值TPS超3000，年增⻓50%，未来5年期望⽀撑量级1万+；
综合以上分8库，部署2个集群，每个集群4个库

1、如果某个sku_id的库存扣减过热，单台实例支撑不了（mysql官方测评：一般单行更新的QPS在500以内），可以考虑将一个sku的大库存拆分成N份，放在不同的库中（也就是说所有子库的库存数总和才是一件sku的真实库存），由于前台的访问流量非常大，按照均分原则，每个子库分到的流量应该差不多。上层路由时只需要在sku_id后面拼接一个范围内的随机数，即可找到对应的子库，有效减轻系统压力。

2、单条sku库存记录更新过热，也可以采用批量提交方式，将多次扣减累计计数，集中成一次扣减，从而实现了将串行处理变成了批处理，也可以大大减轻数据库压力。

3、引入RocketMQ消息队列，经过前置校验后，如果有剩余库存，则把创建订单的操作封装成消息发送给MQ，订单系统从RocketMQ中以特定的频率消费，创建订单，该方案有一定的延迟性。
