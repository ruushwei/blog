---
title: spring
date: 2021-07-29 11:10:43
---

<!-- toc -->

### - 谈谈对springIoc的了解

   [如何完美阐述你对IOC的理解？](https://www.zhihu.com/question/313785621 "知乎")
 
   // 最底层map: Map<String, Object> singletonObjects = new ConcurrentHashMap(256);  

   个人理解：    
   1.控制反转是一种设计思想，将对象生命周期和依赖的控制由对象转给了ioc容器，对象由主动创建其它对象变为被动由ioc容器注入  
   2.springioc容器控制了对象的创建、销毁及整个生命周期，，可以动态注入依赖的对象  
   3.将对象与对象间解耦，对象不再直接创建依赖其它对象，都依赖springioc容器  

   扩展：[springIoc的初始化过程](../2019/09/10/spring/springioc初始化 "springIoc的初始化过程")

它负责业务对象的构建管理和业务对象间的依赖绑定。

### - 谈谈对springAop的了解  

   AOP为面向切面编程，底层是通过动态代理实现，实质上就是将相同逻辑的重复代码横向抽取出来, 拦截对象方法，对方法进行改造、增强！比如在 **方法执行前**、**方法返回后**、**方法前后**、**方法抛出异常后** 等地方进行一定的增强处理，应用场景： 事务、日志、权限、监控打点

   基于动态代理来实现，在容器启动的时候生成代理实例。默认地，如果使用接口的，用 JDK 提供的动态代理实现，如果没有接口，使用 CGLIB 实现

   优点：每个关注点现在都集中于一处，而不是分散到多处代码中；服务模块更简洁，服务模块只需关注核心代码  

   @Pointcut(切点)：用于定义哪些方法需要被增强或者说需要被拦截  
   @Before、@After、@Around、@AfterReturning、@AfterThrowing（增强）: 添加到切点指定位置的一段逻辑代码

   注：@AfterReturning 比 @After 的方法参数多被代理方法的返回值

   参考：  
   [Spring AOP 使用介绍，从前世到今生](https://javadoop.com/post/spring-aop-intro "知乎")  
   [Spring AOP就是这么简单啦](https://juejin.im/post/5b06bf2df265da0de2574ee1 "掘金") 

### - 动态代理和静态代理的区别

静态代理在运行之前就已经存在代理类的字节码文件了（.class文件），而动态代理是在运行时通过反射技术来实现的

**静态代理，如果不同接口的类想使用代理模式来实现相同的功能，将要实现多个代理类**，但在**动态代理中，只需要一个代理类就好了**。

只有实现了某个接口的类可以使用Java动态代理机制，对于没有实现接口的类，就不能使用该机制。使用cglib。

静态代理：代理类与委托类实现同一接口，并且在代理类中需要硬编码接口

JDK动态代理：代理类与委托类实现同一接口，主要是通过代理类实现InvocationHandler并重写invoke方法来进行动态代理的，在invoke方法中将对方法进行增强处理

CGLIB动态代理：代理类将委托类作为自己的父类并为其中的非final委托方法创建两个方法，一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法；另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理


### - Java反射的原理   
反射就是在运行时才知道要操作的类是什么，并且可以在运行时获取类的完整构造，并调用对应的方法    

Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能就称为反射机制。

通俗一点反射就是把Java类中的各种成分映射成一个个的Java对象。

操作步骤:    
1. 获取类的 Class 对象实例
2. 根据 Class 对象实例获取 Constructor 对象
3. 使用 Constructor 对象的 newInstance 方法获取反射类对象
4. 获取方法的 Method 对象
5. 利用 invoke 方法调用方法

为什么类可以动态的生成？
这就涉及到Java虚拟机的类加载机制了，推荐翻看《深入理解Java虚拟机》7.3节 类加载的过程。
Java虚拟机类加载过程主要分为五个阶段：加载、验证、准备、解析、初始化。其中加载阶段需要完成以下3件事情：

通过一个类的全限定名来获取定义此类的二进制字节流
将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据访问入口

关于第1点，获取类的二进制字节流（class字节码）就有很多途径   
动态代理就是想办法，根据接口或目标对象，计算出代理类的字节码，然后再加载到JVM中使用。但是如何计算？如何生成？情况也许比想象的复杂得多，我们需要借助现有的方案


通常使用反射，是想灵活的获取或操作obj的属性或方法信息，而不和具体的某个类耦合。这个时候，只要拿到类的Class对象，就可以获取对象的属性，和执行方法，或者new一个instance。eg: excel导出，<T> 泛型属性解析。


### - spring中的设计模式

#### 简单工厂模式
简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类。 
spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。

#### 工厂方法模式
工厂方法将创建产品的代码与实际使用产品的代码分离， 从而能在不影响其他代码的情况下扩展产品创建部分代码。
例如， 如果需要向应用中添加一种新产品， 你只需要开发新的创建者子类， 然后重写其工厂方法即可。

一般情况下,应用程序有自己的工厂对象来创建bean.如果将应用程序自己的工厂对象交给Spring管理, 那么Spring管理的就不是普通的bean,而是工厂Bean。
理解：通过扩展工厂子类的方式，调用其工厂方法创建新对象。

#### 单例模式
保证一个类仅有一个实例，并提供一个访问它的全局访问点。 
spring中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory。但没有从构造器级别去控制单例，这是因为spring管理的是是任意的java对象。 

Spring下默认的bean均为singleton，可以通过singleton=“true|false” 或者 scope="?"来指定。

#### 代理设计模式 
Spring AOP 功能的实现。

#### 适配器模式
springaop中的advice，需要适配成 MethodInterceptor 接口，才能组成一条拦截器链，做依次的织入。
不同的Advice需要通过 不同适配器（XXXAdviceInterceptor） 适配成一个个 MethodInterceptor。使用适配器模式，持有advice实例，实现MethodInterceptor 接口。

我们知道 Spring AOP 的实现是基于代理模式，但是 Spring AOP 的增强或通知(Advice)使用到了适配器模式，与之相关的接口是AdvisorAdapter 。Advice 常用的类型有：BeforeAdvice（目标方法调用前,前置通知）、AfterAdvice（目标方法调用后,后置通知）、AfterReturningAdvice(目标方法执行结束后，return之前)等等。
每个类型Advice（通知）都有对应的拦截器:MethodBeforeAdviceInterceptor、AfterReturningAdviceAdapter、AfterReturningAdviceInterceptor。
Spring预定义的通知要通过对应的适配器，适配成 MethodInterceptor接口(方法拦截器)类型的对象（如：MethodBeforeAdviceInterceptor 负责适配 MethodBeforeAdvice）。

参考文档：https://www.jianshu.com/p/57d3d1beeb62

#### 模板方法模式
JDBCTemplate使用了模板方法 + 回调模式。把直接执行statment的操作延迟到子类实现，通过回调函数的方式传递进来，父类实现对连接和异常的处理等操作。
execute中接收一个 StatementCallback 的回调函数，子类只要传递自己处理statment获取结果的回调函数即可，不需要再写过多的重复代码。

```java
@FunctionalInterface
public interface StatementCallback<T> {
   @Nullable
   T doInStatement(Statement var1) throws SQLException, DataAccessException;
}
```

execute实现（调用回调函数StatementCallback#doInStatement）
```java
@Nullable
public <T> T execute(StatementCallback<T> action) throws DataAccessException {
   //参数检查
   Assert.notNull(action, "Callback object must not be null");
   //获取连接
   Connection con = DataSourceUtils.getConnection(this.obtainDataSource());
   Statement stmt = null;
   Object var11;
   try {
      //创建一个Statement
      stmt = con.createStatement();
      //设置查询超时时间，最大行等参数（就是一开始那些成员变量）
      this.applyStatementSettings(stmt);
      //执行回调方法获取结果集
      T result = action.doInStatement(stmt);
      //处理警告
      this.handleWarnings(stmt);
      var11 = result;
   } catch (SQLException var9) {
      //出现错误优雅退出
      String sql = getSql(action);
      JdbcUtils.closeStatement(stmt);
      stmt = null;
      DataSourceUtils.releaseConnection(con, this.getDataSource());
      con = null;
      throw this.translateException("StatementCallback", sql, var9);
   } finally {
      JdbcUtils.closeStatement(stmt);
      DataSourceUtils.releaseConnection(con, this.getDataSource());
   }
   return var11;
}
```

update方法详解（使用lambda函数传递更新的实现操作）

```java
protected int update(PreparedStatementCreator psc, @Nullable PreparedStatementSetter pss) throws DataAccessException {
   this.logger.debug("Executing prepared SQL update");
   return updateCount((Integer)this.execute(psc, (ps) -> {
      Integer var4;
      try {
            if (pss != null) {
               pss.setValues(ps);
            }
            int rows = ps.executeUpdate();
            if (this.logger.isTraceEnabled()) {
               this.logger.trace("SQL update affected " + rows + " rows");
            }
            var4 = rows;
      } finally {
            if (pss instanceof ParameterDisposer) {
               ((ParameterDisposer)pss).cleanupParameters();
            }
      }
      return var4;
   }));
   }
```

JDBCTemplate使用了很多回调。为什么要用回调（Callback)?
如果父类有多个抽象方法，子类需要全部实现这样特别麻烦，而有时候某个子类只需要定制父类中的某一个方法该怎么办呢？这个时候就要用到Callback回调了就可以完美解决这个问题，可以发现JDBCTemplate并没有完全拘泥于模板方法，非常灵活。我们在实际开发中也可以借鉴这种方法。

参考文档：https://juejin.cn/post/6844903847966703624#heading-5


### - spring aspect advice pointcut理解
1、adivisor是一种特殊的Aspect，Advisor代表spring中的Aspect 
2、区别：advisor只持有一个Pointcut和一个advice，而aspect可以多个pointcut和多个advice

- 方/切 面（Aspect）：一个关注点的模块化，这个关注点实现可能另外横切多个对象。事务管理是J2EE应用中一个很好的横切关注点例子。方面用Spring的Advisor或拦截器实现。 
- 连接点/织入点（Joinpoint）：程序执行过程中明确的点，如方法的调用或特定的异常被抛出。 
- 通知（Advice）：在特定的连接点，AOP框架执行的动作。各种类型的通知包括“around”、“before”和“throws”通知。 
- 切入点（Pointcut）：指定一个通知将被引发的一系列连接点的集合。AOP框架必须允许开发者指定切入点，例如，使用正则表达式。

#### Springboot启动原理
springboot在启动时除了spring context容器启动的流程，还加入了通过spi实现的根据依赖自动装配的机制。
springboot容器启动的流程，先初始化事件监听器，加载环境信息，创建applicationContext容器，执行applicationInitializer的initialize方法，在容器refresh时，会通过spi机制获取到所有的autoConfiguration类（解析spring.factotries文件）并加载配置类中相关bean注入context容器，完成自动装配。

#### Springioc解决循环依赖问题
使用中间缓存，使用二级缓存earlySingletonObjects，三级缓存singletonFactories一起解决的循环依赖问题。
earlySingletonObjects就是一个临时存放初始bean的缓存。
singletonFactories是会在获取依赖的A时，调用一个wrapIfNessasory来获得一个aop后的A, 从而解决循环依赖aop bean的问题。
注：springioc只能解决set注入的循环依赖问题，无法解决构造方法注入的循环依赖问题，因为A在构造时依赖B，B在构造获取A的时候，A都还没有创建出来，没有能放到一个中间缓存解决循环依赖的机会。
