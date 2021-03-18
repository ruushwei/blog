---
title: pcursor基于游标的分页
date: 2020-08-13T12:54:24+02:00
tags: 
- pcursor
- design
categories: design
---

<!-- toc -->

### 一、基于偏移量分页&基于游标分页

#### 1.基于偏移量的分页

适用于内容是静态的，或者不用实时返回数据最新的变化。特点：1.只要有新增或删除，就会有大量的数据偏移量的改变，造成重复展示或者漏展示。2.存在数据量大时的offset慢查询问题

![img](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/clipboard_20200814041307.png)



#### 2.基于游标的分页

适用于查询结果在用户浏览过程中是变化的。特点：1.时间线里出现新的数据或删除数据，这些变化都可以在 “前一页” 或者 “后一页” 体现出来。 2.没有offset问题

eg: 发现页 下拉获取最新，上拉获取更久

### 二、接口实现

请求参数

| 参数名  | 类型    | 逻辑                               |
| :------ | :------ | :--------------------------------- |
| 参数名  | 类型    | 逻辑                               |
| pcursor | String  | 初始值传 "" , 后续值使用后端返回值 |
| count   | Integer | 本次想获取的数据量                 |

响应信息

| 参数名  | 类型   | 逻辑                                                         |
| :------ | :----- | :----------------------------------------------------------- |
| pcursor | String | 透传给下一页查询 调用方判断是否为 "no_more", 若是则不再调用，否则永远停不下来了 |

### 三、底层实现

根据更新时间排序

排序规则：order by update_time desc, id desc  原因: 更新时间 + id 的排序, 可以为所有数据确定顺序，即使更新时间相同，即确定每条记录的游标，游标为更新时间+id，只要查询条件不变，即游标可在该顺序下唯一确定一个位置

#### 1. pcursor游标如何拼接

- 根据上面排序规则查询出数据
- 取最后一个数据last，即为该排序下此次查询的终止位置
- 根据last数据拼接成pcursor
  pcursor = encrypt ( ("updateTime", long, 1589439219430); ("id", long, 95424 ); )

#### 2.pcursor游标如何使用

- pcursor解析为 上次查询last的 last.updateTime, last.id 
- 本次查询where条件中，updateTime < last.updateTime or (update_time = last.updateTime and id < last.id) order by update_time desc, id desc;    
  如下表格，为order by update_time desc, id desc，上次查询游标为last游标位置，则下次查询 updateTime < 1555500001 or (updateTime = 1555500001 and id < 32) order by update_time desc, id desc， 即为后三行数据

| update_time | id   |               |
| :---------- | :--- | :------------ |
| 1555500001  | 33   |               |
| 1555500001  | 32   | last 游标位置 |
| 1555500001  | 31   |               |
| 1555500000  | 44   |               |
| 1555500000  | 42   |               |

#### 3. 生成查询SQL

若查询条件为 userId, status, updateTime 则查询语句为

```
SELECT *
FROM
    ad_xxx_info
WHERE
    user_id = xxx
    AND STATUS = xxx
    AND (
        update_time < xxx
        OR ( update_time = xxx AND id < xxx )
    )
ORDER BY
    update_time DESC, id DESC
LIMIT 20;
```

创建联合索引 (user_id, status, update_time) 查询即可全部走索引。注：主键也是索引的一部分，Extra为Using where。using where is fine https://stackoverflow.com/questions/9533841/mysql-fix-using-where