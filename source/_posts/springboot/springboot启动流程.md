---
title: springboot启动流程
date: 2021-09-23 15:10:43
tags: 
- springboot
categories: springboot
---

springboot 启动原理
参考文档：https://mp.weixin.qq.com/s/1yHtaoSqqIJItIXByvV5QA

#### @SpringBootApplication注解与@EnableAutoConfiguration与AutoConfigurationImportSelector.class 与 SpringFactoriesLoader
@SpringBootApplication = @Configuration+ @EnableAutoConfiguration+ @ComponentScan
##### @Configuration
JavaConfig形式的Spring Ioc容器的配置类，等同于xml方式的 <beans/>
##### @ComponentScan: 
自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。

##### @EnableAutoConfiguration
借助@Import的支持，收集和注册特定场景相关的bean定义。
@AutoConfigurationPackage + @Import(AutoConfigurationImportSelector.class)

AutoConfigurationPackage将 添加该注解的类所在的package 作为 自动配置package 进行管理。

@Import(EnableAutoConfigurationImportSelector.class)，借助EnableAutoConfigurationImportSelector，@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持。实现智能的自动配置。

SpringFactoriesLoader：
SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置。
配合@EnableAutoConfiguration使用的话，它更多是提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key，获取对应的一组@Configuration类。

so:
@EnableAutoConfiguration自动配置的魔法骑士就变成了：从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。

#### Springboot启动执行详细流程
1. SpringApplication初始化
A.根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。
B.使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationContextInitializer。
C.使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationListener。
D.推断并设置main方法的定义类。

2.  SpringApplication run()
A.加载所有SpringApplicationRunListener。调用它们的started()方法
`` SpringApplicationRunListeners listeners = getRunListeners(args);
`` 		listeners.starting(bootstrapContext, this.mainApplicationClass);
B. 创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。
`` ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
`` 			configureIgnoreBeanInfo(environment);
C.遍历调用所有SpringApplicationRunListener的environmentPrepared()的方法，告诉他们：“当前SpringBoot应用使用的Environment准备好了咯！”
D. 如果SpringApplication的showBanner属性被设置为true，则打印banner
E. 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成
`` context = createApplicationContext();
F. 将之前准备好的Environment设置给创建好的ApplicationContext使用, 将遍历调用ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理
`` context.setEnvironment(environment);
`` 		postProcessApplicationContext(context);
`` 		applyInitializers(context);
G. 遍历调用所有SpringApplicationRunListener的contextPrepared()方法。 
`` listeners.contextPrepared(context);
H. 注册特殊定义bean（springApplicationArguments、springBootBanner）, 添加LazyInitializationBeanFactoryPostProcessor，创建并初始化BeanDefinitionLoader（主要做的工作就是注册bean）
I.  遍历调用所有SpringApplicationRunListener的contextLoaded()方法。
`` listeners.contextLoaded(context);
J. 调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序.
`` applicationContext.refresh();
K.执行ApplicationRunners，CommandLineRunners
`` callRunners(context, applicationArguments);
L.遍历执行SpringApplicationRunListener的started()方法


#### Springboot启动简要流程
1. SpringApplication 初始化，收集所有ApplicationContextInitializer和ApplicationListener。
2. SpringApplication run
    调用started事件
    创建并准备Environment（PropertySource、Profile）
    调用environmentPrepared事件
    创建ApplicationContext，遍历调用ApplicationContextInitializer的initialize方法
    调用contextPrepared 事件
    创建并配置BeanDefinitionLoader
    调用contextLoaded 事件
    applicationContext的refresh()方法, 完成ioc容器初始化
    调用started事件
    callRunners（ApplicationRunners，CommandLineRunners）
    调用running事件


#### 总结
Springboot的启动，主要创建了配置环境(environment)、事件监听(listeners)、应用上下文(applicationContext)，并基于以上条件，在容器中开始实例化我们需要的Bean
自动装配核心：@EnableAutoConfiguration中的AutoConfigurationImportSelector中的SpringFactoriesLoader 提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key，获取对应的一组@Configuration类。并在后续加载到ioc容器。