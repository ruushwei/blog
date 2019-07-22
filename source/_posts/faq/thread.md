---
title: 多线程相关问答
date: 2019-07-01T12:54:24+02:00
tags: 
- 面试
- 线程
categories: 线程
---

<!-- toc -->

#### 阿里开发手册上写着关于线程创建的问题  

【强制】线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。  

说明：Executors返回的线程池对象的弊端如下：

1）CachedThreadPool 和 ScheduledThreadPool :  
允许创建的线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。

2）FixedThreadPool 和 SingleThreadPool：   
允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。


#### Executors的四个线程池构造方法

Executors.newCachedThreadPool();
```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

Executors.newScheduledThreadPool(1);
```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
    }
```

*CachedThreadPool 和 ScheduledThreadPool :  
允许创建的线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。*


Executors.newSingleThreadExecutor();
```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```


Executors.newFixedThreadPool();
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

*FixedThreadPool 和 SingleThreadPool：   
允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。*


#### ThreadPoolExecutor都有哪些参数

核心线程数， 最大线程数，非核心线程空闲存活时间，阻塞队列，拒绝策略，有必要的话还有ThreadFactory线程工厂.


#### 使用ThreadPoolExecutor创建线程池示例   
```java
    private static int corePoolSize = 5;
    private static int maximumPoolSize = 100;
    private static long keepAliveTime = 60L;
    private static ExecutorService executorService = new ThreadPoolExecutor(
            corePoolSize,      // 核心线程
            maximumPoolSize,   // 最大线程数，在队列满的时候，进行扩容的最大容量  （core、max 可以更好的利用线程资源）
            keepAliveTime,     // 非核心线程数，空闲回收时间  （增加线程利用，减少线程重复创建）
            TimeUnit.SECONDS,  // 空闲回收时间的单位
            new ArrayBlockingQueue(10),    // 阻塞队列 （设置队列大小，很关键）
            new ThreadPoolExecutor.AbortPolicy());  // 当队列已经满了，执行的处理
```
还有一个可传参数是 ThreadFactory threadFactory, 我们通常使用 Executors.defaultThreadFactory()



#### 使用ThreadPoolExecutor的好处

参数带来的好处, 线程池就是为了更好更充分的利用线程资源，线程资源是宝贵的

相比Executors创建线程，ThreadPoolExecutor 可定制化线程池，创建符合场景下的线程池。可以通过参数配置避免Executors的OOM

1. corePoolSize, maximumPoolSize 弹性控制线程数量，可伸缩，可扩容可释放
2. keepAliveTime, TimeUnit.SECONDS 设置后在回收前可让其他任务使用，减少重新创建线程的开销
3. BlockingQueue, 设置队列大小，起到缓冲的作用
4. 自定义拒绝策略

总结： 弹性线程数、增加空闲线程再利用、队列缓冲任务量、自定义拒绝策略


#### 线程池原理

把一个任务放入线程池的execute()函数中，线程池会为我们选择一个线程来执行我们提交的任务。在这个选择线程的过程中，如果线程池中线程数量小于corePoolSize，那么将创建新线程执行任务；当线程池数量大于等于corePoolSize并且小于maximumPoolSize，线程池会把任务放到阻塞队列workQueue中直到workQueue满了去创建新线程；当线程池线程数量等于maximumPoolSize并且workQueue满时会执行拒绝策略


#### 四种拒绝策略

##### AbortPolicy
ThreadPoolExecutor中默认的拒绝策略就是AbortPolicy。直接抛出异常

```java
public static class AbortPolicy implements RejectedExecutionHandler {
    public AbortPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}
```

##### CallerRunsPolicy
CallerRunsPolicy在任务被拒绝添加后，会调用当前线程池的所在的线程去执行被拒绝的任务。

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    public CallerRunsPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}
```

##### DiscardPolicy
这个策略的处理就更简单了，看一下实现就明白了：

```java
public static class DiscardPolicy implements RejectedExecutionHandler {
    public DiscardPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
```

##### DiscardOldestPolicy
DiscardOldestPolicy策略的作用是，当任务呗拒绝添加时，会抛弃任务队列中最旧的任务也就是最先加入队列的，再把这个新任务添加进去。

```java
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    public DiscardOldestPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```