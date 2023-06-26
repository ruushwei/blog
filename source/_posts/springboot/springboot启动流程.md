---
title: springboot启动原理
date: 2023-06-10 15:10:43
tags: 
- springboot
categories: springboot
---

### 一句话
springboot在启动时除了spring context容器启动的流程，还加入了通过spi实现的根据依赖自动装配的机制。
springboot容器启动的流程，先初始化事件监听器，加载环境信息，创建applicationContext容器，执行applicationInitializer的initialize方法，在容器refresh时，会通过spi机制获取到所有的autoConfiguration类（解析spring.factotries文件）并加载配置类中相关bean注入context容器，完成自动装配。

### 细分原理
#### 应用启动，容器初始化
1、通过spi加载所有的springApplicationRunListener，在后续各个启动步骤中发消息
2、创建环境Environment，加载属性文件配置、加载命令行参数等启动时所需的配置信息
3、创建ApplicationContext,将环境Environment设置给context
4、通过spi加载所有的applicationInitializer并调用其initialize方法
5、【核心】将spi拿到的EnableAutoConfiguration配置类的所有bean加载到已创建好的ioc容器
6、调用applicationContext的refresh方法，完成ioc容器可用

问题1：创建的环境Environment里都有什么？
加载属性文件配置、加载命令行参数等启动时所需的配置信息

问题2：spi拿到全部EnableAutoConfiguration类的具体入口？
refreshContext方法，bean实例化之前，执行AutoConfigurationImportSelector的selectImports方法，返回要实例化的类信息列表。

问题3：applicationContext的refresh方法主要逻辑？
初始化环境、设置类加载器 -> 解析加载BeanDefinition对象(应该是这里拿到的全部的EnableAutoConfiguration类) -> 将BeanDefinition对象注册到BeanFactory -> 实例化非懒加载的单例Bean -> 注册所有BeanPostProcessor到BeanFactory中 -> 初始化剩余非懒加载的Bean初始化 -> 完成刷新操作，发布容器事件 -> 返回刷新后的BeanFactory

#### 自动装配
SPI(service provider interface), 定义接口后，通过读取文件配置的方式加载实现类。
关键注解:@EnableAutoConfiguration, 借助@Import，SpringFactoriesLoader, 把所有spring.factories文件配置的@Configuration配置类找到并实例化，将其中bean加载到ioc容器，即实现了自动装配。