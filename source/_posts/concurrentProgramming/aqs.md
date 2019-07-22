---
title: AQS原理解析
date: 2019-06-02T12:54:24+02:00
tags: 
- java
- 并发
categories: 并发
---

#### 谈谈你对AQS的理解

AQS是JUC(java.util.concurrent)中很多同步组件的构建基础，简单来讲，它内部实现主要是状态变量state和一个FIFO队列来完成，同步队列的头结点是当前获取到同步状态的结点，获取同步状态state失败的线程，会被构造成一个结点（或共享式或独占式）加入到同步队列尾部（采用自旋CAS来保证此操作的线程安全），随后线程会阻塞；释放时唤醒头结点的后继结点，使其加入对同步状态的争夺中。

##### AQS的内部实现

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/1524404680870b808dc51a9.jpeg)


- state 资源状态
- exclusiveOwnerThread 持有资源的线程
- CLH 同步等待队列。

AQS维护一个共享资源state，内置的同步队列由一个一个的Node结点组成，每个Node结点维护一个prev引用和next引用，分别指向自己的前驱和后继结点。

AQS维护两个指针，分别指向队列头部head和尾部tail


##### 继承自AQS的常见类

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/WechatIMG33.jpeg)

CountDownLatch、ReentrantLock、Semaphore等


##### AQS的两种实现方式

独占式&共享式, 这样方便使用者实现不同类型的同步组件，独占式如ReentrantLock，共享式如Semaphore，CountDownLatch，组合式的如ReentrantReadWriteLock。

AQS定义的以下可重写的方法：

```
独占式获取同步状态，试着获取，成功返回true，反之为false
protected boolean tryAcquire(int arg) 

独占式释放同步状态，等待中的其他线程此时将有机会获取到同步状态
protected boolean tryRelease(int arg) 
  
共享式获取同步状态，返回值大于等于0，代表获取成功；反之获取失败
protected int tryAcquireShared(int arg) 

共享式释放同步状态，成功为true，失败为false
protected boolean tryReleaseShared(int arg) 

是否在独占模式下被线程占用
protected boolean isHeldExclusively()
```


**独占式**

1.tryAcquire方法尝试获取锁，如果成功就返回，如果不成功，走到2.

2.把当前线程和等待状态信息构造成一个Node节点，并将结点放入同步队列的尾部

3.该Node结点在队列中尝试获取同步状态，若获取不到，则阻塞结点线程，直到被前驱结点唤醒或者被中断.

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的

自我实现独占锁示例，待重写方法提供更新state等操作.

```java
import java.util.concurrent.locks.AbstractQueuedSynchronizer;

public class MyLock {


    private final Sync sync = new Sync();

    public void lock() {
        sync.acquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }


    private static class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire(int arg) {

            //首先判断状态是否等于=0,如果状态==0，就将status设置为1
            if (compareAndSetState(0,1)) {
                //将当前线程赋值给独占模式的onwer
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }

            return false;

        }

        @Override
        protected boolean tryRelease(int arg) {
            if (getState() == 0) {
                throw new IllegalMonitorStateException();
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }
    }
}
```

**共享式**

CountDownLatch:  
任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

Semaphore:  
AQS通过state值来控制对共享资源访问的线程数，有线程请求同步状态成功state值减1，若超过共享资源数量获取同步状态失败，则将线程封装共享模式的Node结点加入到同步队列等待。有线程执行完任务释放同步状态后，state值会增加1，同步队列中的线程才有机会获得执行权。公平锁与非公平锁不同在于公平锁申请获取同步状态前都会先判断同步队列中释放存在Node，若有则将当前线程封装成Node结点入队，从而保证按FIFO的方式获取同步状态，而非公平锁则可以直接通过竞争获取线程执行权。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。