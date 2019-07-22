---
title: 项目相关问答
date: 2019-07-01T12:54:24+02:00
tags: 
- 面试
- 项目
categories: 项目
---
### 项目介绍

#### 选x服务

简介： xxx

一、迁移Java   
从无到有 C++ 到 java ， 修改全量更新为增量更新策略，考虑younggc
迁移后单机可承受1wqps，保持4个9 30ms  

二、性能优化
1. 代码层面，减少load数据，减少younggc
2. 启动时多次fullgc （metaspace扩容导致fullgc），younggc严重（load数据）， 采取初始化完显示System.gc() 减少启动后的GC波动
3. 凌晨有时promotion failed 导致穿行fullgc, 调整 CMSInitiatingOccupancyFraction
4. 整点younggc， 分页更新，且将整点敏感的活动推动提前预热比如秒杀
5. 一些其他为优化gc的小调整，机器扩容，新生代比例调整

重要点：  
解决GC问题，提供稳定服务，解决启动、凌晨、整点的gc问题
从迁移开发到维护，确保索引的准确性与服务可用性

#### 库x服务

简介：xxx

直减库存模型的统一，db、tair ，迁移新接口
支持拼x库存，二级库存，支持从拼x工具调用为团x锁定整个团活x库x等操作
支持下单是否锁定库x，活x维度控制
提供商家库x限制、抵用x优惠计x等能力


活动库x限制应该是一个表，stock， 里面有userid ， dealid， groupid ，partnerid 维度做限制，都是二级库x维度，如果id = 0 ， 则是所有用户或所有团都限制，如果是确定的某个值，比如partnerid，则是对某商x库x的限制， 其他商x 是id=0 行对应的次数限制，如果是0，则是都不能使，如果是有值，则其他商x都是该值，相当于可通用，可定制

活x锁定库x在lock表，活x核销库x在lockrecord表。 tair 做分布式锁，会有多线程脚本做库x校正。

### 项目中比较有难度的是哪里？ (amazon, caocao)

GC

### 项目中比较有哪些觉得需要重构的地方？(amazon)

xxx索引部分