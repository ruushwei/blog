---
title: tair & redis 的选择
date: 2019-07-09T12:54:24+02:00
tags: 
- redis
- tair
categories: redis
---


#### tair & redis 的选择

1. 延迟敏感程度, 延迟敏感，说什么也得redis  
2. 数据量大的话，数据量超过100GB全内存太浪费资源，延迟没有那么敏感，使用tair  
3. 使用复杂数据结构 redis  
4. 容忍数据丢失  
