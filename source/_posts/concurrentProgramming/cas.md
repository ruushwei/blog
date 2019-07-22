---
title: CAS原理、缺点、AtomicInteger使用解析
date: 2019-05-29T12:54:24+02:00
tags: 
- jvm
- 并发
categories: 并发
---



#### cas原理

CAS（Compare and Swap），即比较并替换, 是设计并发算法时用到的一种技术。简单来说，比较和替换是使用一个期望值和一个变量的当前值进行比较，如果当前变量的值与我们期望的值相等，就使用一个新值替换当前变量的值。

CAS有三个操作数：内存值V、旧的预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做并返回false。

在CAS中，比较和替换是一组原子操作，不会被外部打断，且在性能上更占有优势。

<!-- more-->

####  Unsafe提供了三个方法用于CAS操作

```java

public final native boolean compareAndSwapObject(Object value, long valueOffset, Object expect, Object update);

public final native boolean compareAndSwapInt(Object value, long valueOffset, int expect, int update);

public final native boolean compareAndSwapLong(Object value, long valueOffset, long expect, long update);

```

#### 解析AtomicInteger

当我们使用 AtomicInteger 实现多线程的加操作时，分析源码

```java

public class AtomicInteger extends Number implements java.io.Serializable {

​    // setup to use Unsafe.compareAndSwapInt for updates

​    private static final Unsafe unsafe

​         = Unsafe.getUnsafe();

​    private static final long valueOffset;

​    static {

​        try {

​            valueOffset = unsafe.objectFieldOffset(AtomicInteger.class.getDeclaredField(

​                        "value"));

​        }

​        catch (Exception ex) {

​            throw new Error(ex);

​        }

​    }

​    private volatile int value;

​    public final int get() {

​        return value;

​    }

}

public final int getAndAdd(int delta) {

​    return unsafe.getAndAddInt(this, valueOffset, delta);

}

// 重点

public final int getAndAddInt(Object var1, long var2, int var4) {

​    int var5;

​    do {

​        var5 = this.getIntVolatile(var1, var2);

​    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

​    return var5;

}

```

#### 解析

1. Unsafe，是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。

2. 变量valueOffset，表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的。

3. 变量value用volatile修饰，保证了多线程之间的内存可见性. 我们在使用cas更新value的时候，没有用到volatile，但get、set方法获取value时用到了volatile的可见性。这样结合来看使得AtomicInteger类获取的是主存中最新的值 且 更新时cas是原子性操作。

4. 假设线程A和线程B同时执行getAndAdd操作, 因为比较与替换是原子性操作，所以即使在var5获取到相同的值，也会是顺序的更新。

#### CAS缺点

1. ABA问题。

   如果在这段期间曾经被改成B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。针对这种情况，java并发包中提供了一个带有标记的原子引用类 AtomicStampedReference，它可以通过控制变量值的版本来保证CAS的正确性, 该类提供一个引用和计数变量。

2. 循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

3. 只能保证一个共享变量的原子操作

   当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性

   从Java1.5开始JDK提供了 AtomicReference 类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作