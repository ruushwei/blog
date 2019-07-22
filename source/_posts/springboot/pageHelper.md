---
title: pageHelper-springboot
tags:
- springboot
- mybatis
date: 2018-02-06 13:23:22
categories: springboot
---





springboot Mybatis 数据库分页获取 ，使用pageHelper插件



 ### maven

```xml
<!--pagehelper-->
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper-spring-boot-starter</artifactId>
	<version>1.1.1</version>
</dependency>
```



引用此依赖可能会与其它springboot依赖冲突，此时可手动去除其它依赖

```xml
compile ('com.github.pagehelper:pagehelper-spring-boot-starter:1.2.5') {
    exclude group: 'org.springframework.boot', module: ''
    exclude group: 'org.mybatis.spring.boot', module: ''
}
```



### application.properties

```properties
#pagehelper分页插件配置
pagehelper.helperDialect=mysql
pagehelper.reasonable=true
pagehelper.supportMethodsArguments=true
pagehelper.params=count=countSql
```



### 调用

```java
PageHelper.startPage(pageNum, pageSize);
mapper.loadAll();
```

<!--more-->