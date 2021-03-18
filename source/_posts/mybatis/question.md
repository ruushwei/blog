---
title: mybatis疑问解答
date: 2020-08-07T12:54:24+02:00
tags: mybatis
categories: mybatis
---

### Mybatis的点

通过动态代理，自动生成一些参数的映射、sql的执行、结果的映射，减少了大量重复的代码，且对sqlsession，jdbc操作，事务，一二级缓存，做了很好的封装

### MyBatis的工作原理

 1）读取 MyBatis 配置文件：mybatis-config.xml 为 MyBatis 的全局配置文件，配置了 MyBatis 的运行环境等信息，例如数据库连接信息。   
 2）加载映射文件。映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件 mybatis-config.xml 中加载。mybatis-config.xml 文件可以加载多个映射文件，每个文件对应数据库中的一张表。    
 3）构造会话工厂：通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。   
 4）创建会话对象：由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。   
 5）Executor 执行器：MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。    
 6）MappedStatement 对象：在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息。   
 7）输入参数映射：输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型。输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程。    
 8）输出结果映射：输出结果类型可以是 Map、 List 等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程。    


### mybaits中如何维护一级缓存、一级缓存的生命周期、一级缓存何时失效

BaseExecutor成员变量之一的PerpetualCache，是对Cache接口最基本的实现， 其实现非常简单，内部持有HashMap，对一级缓存的操作实则是对HashMap的操作。

一级缓存的生命周期和SqlSession一致，即一次会话

何时失效   
a. MyBatis在开启一个数据库会话时，会创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象，Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。   
b. 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用；   
c. 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用；   
d.SqlSession中执行了任何一个update操作update()、delete()、insert() ，都会清空PerpetualCache对象的数据   

### 一级缓存的工作流程？

a.对于某个查询，根据statementId,params,rowBounds来构建一个key值，根据这个key值去缓存Cache中取出对应的key值存储的缓存结果；   
b. 判断从Cache中根据特定的key值取的数据数据是否为空，即是否命中；   
c. 如果命中，则直接将缓存结果返回；   
d. 如果没命中：去数据库中查询数据，得到查询结果；将key和查询到的结果分别作为key,value对存储到Cache中；将查询结果返回.   


### mybatis sqlsession、connection、transition的理解

一个sqlsession会话，如果是多个更新操作   

自动提交： 代码层是一个connection、一个transition事务配置， db层是一个connection，多个事务

手动提交： 代码层是一个connection、一个transition事务配置，db层也是一个connection，一个事务


### Mybatis动态sql是做什么的？都有哪些动态sql？简述一下动态sql的执行原理？

动态sql是用一定规则方便的拼接sql。

动态sql: <where>、<if>  <foreach>

执行原理：获取当前节点的子节点，判断子节点的类型是否为文本或者CDATA，如果是则没有动态sql，如果节点类型为元素，则表明是动态sql，根据动态sql标签类型，给到不同的handler做处理，最终拼成一个sql


### Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？

支持。

实现原理是动态代理。在resultSetHandler做结果集映射的时候，判断下属性是否配置了嵌套查询，如果配置了则先创建代理对象，实现代理对象的invoke方法，在对下get操作的时候，再去执行该字段的获取方法。


### Mybatis都有哪些Executor执行器？它们之间的区别是什么？

SimpleExecutor, ReuseExecutor , BatchExecutor , CachingExecutor

SimpleExecutor：每一次执行都会创建一个新的Statement对象.

ReuseExecutor: 重复使用Statement对象

ReuseExecutor比SimpleExecutor 性能更好一些


BatchExecutor：批量statement提交。其事务无法自动提交

CachingExecutor：用来做二级缓存，装饰者模式。

实际生产环境中建议使用 ReuseExecutor 


### 简述下Mybatis的一级、二级缓存（分别从存储结构、范围、失效场景。三个方面来作答）？

存储结构：一级、二级缓存的存储结构都为hashMap

范围：一级缓存的范围是sqlsession级别，二级缓存的范围是namespace级别

失效场景：

当sqlsession做插入、更新、删除操作时


### 简述Mybatis的插件运行原理，以及如何编写一个插件？

拦截器，对mybatis的四大核心对象做拦截，做功能增强，底层动态代理实现，四大对象创建时，都会走到interceptorChain.pluginAll(), 返回增强后的对象

编写插件：
1.实现一个interceptor， 加上注解， @Intercepts, 配置要增强的四大对象的接口和方法名
2.sqlMapperConfig中添加该自定义插件标签


参考文献：   
https://juejin.im/post/6844904101587714061
https://www.shangmayuan.com/a/cfc40f96680e49e9b692a4e7.html