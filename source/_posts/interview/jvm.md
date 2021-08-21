---
title: jvm
date: 2021-07-29 11:10:43
---

<!-- toc -->

### - Java内存区域

 **线程私有的** 

程序计数器
存放当前线程所执行的字节码的行号指示器。如果线程执行的是一个Java程序，计数器记录正在执行的虚拟机字节码指令地址；正在执行的是native方法，则计数器的值为空

Java虚拟机栈

描述Java方法执行的内存模型，每个方法执行的同时都会创建一个栈帧来存储局部变量表（编译期可知的各种基本数据类型，对象引用【reference类型】，内存空间的分配是在编译期完成的，方法运行期间不会改变局部变量表的大小）、操作数栈、动态链接、方法出口等。

本地方法栈

为虚拟机使用到的native方法服务

**线程公有的** 

Java堆（GC堆）

存放对象实例，所有的对象实例和数组都在堆上分配。程序运行时分支可能不一样，只有运行期间才能知道创建哪些对象，这部分内存的分配和回收是动态的

metaspace

存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码

直接内存

不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存。NIO使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作，避免了Java堆和Native堆来回复制数据。直接内存的回收是在虚拟机进行full gc的时候顺带进行的，并不会自己触发垃圾回收

### - 谈谈java的类加载机制

1. 过程   
	- 加载   
		通过全类名获取定义此类的二进制字节流   
		将字节流所代表的静态存储结构转换为方法区的运行时数据结构   
		在内存中生成一个代表该类的 Class 对象,作为方法区这些数据的访问入口   
	- 连接   
		验证   
      文件格式、元数据、字节码、符号引用验证   
	- 准备   
		正式为类变量分配内存并设置类变量初始值的阶段，设置数据类型默认的零值   
	- 解析   
		解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程   
	- 初始化   
		真正执行类中定义的 Java 程序代码   
      初始化阶段是执行类构造器 <clinit> ()方法的过程   
2. 双亲委派模式   
   Java的类加载使用双亲委派模式，即一个类加载器在加载类时，先把这个请求委托给自己的父类加载器去执行，如果父类加载器还存在父类加载器，就继续向上委托，直到顶层的启动类加载器。如果父类加载器能够完成类加载，就成功返回，如果父类加载器无法完成加载，那么子加载器才会尝试自己去加载。这种双亲委派模式的好处，一个可以避免类的重复加载，另外也避免了java的核心API被篡改。

   BootstrapClassLoader  
	ExtensionClassLoader  
	AppClassLoader(应用程序类加载器)
   
### - gc对象存活判定算法

1.1引用计数算法

给对象添加一个引用计数器，当有地方引用它时加1，当引用失效时计数减1，任何时刻计数器为0时的对象都不会再被引用。

缺点：存在两个对象不会被继续访问，但是两者循环调用的情况，由于存在循环调用导致两个对象都不会被回收。

1.2可达性分析算法

通过一系列被称作gc roots对象作为起点，从这些起点向下搜索，搜索所走过的路径成为引用链，当一个对象到gc roots没有任何引用链时证明此对象不可用。

Java中gc roots的对象包括：

虚拟机栈中引用的对象  
native方法中引用的对象   
类静态属性、常量中引用的对象  

对于一个 Java 程序而言，对象都位于堆内存块中，存活的那些对象都被根节点引用着，即根节点 GC Roots 是一些引用类型，自然不在堆里，那它们位于哪呢？它们能放在哪呢？        
答案是放在栈里，包括：    
Local variables 本地变量    
Static variables 静态变量        
JNI References JNI引用等    
    

### - gc垃圾收集器及其优缺点

垃圾回收器种类:Serial、ParNew、Parallel Scavenge、Serial Old、Parallel Old、CMS(ConCurrent Mark Sweep)、G1(Garbage First)

### - 标记算法和复制算法的区別，用在什么场合  
CMS 标记清除、Serial Old，Parallel old 标记整理：适用于存活对象比较多的场景
Serial、ParNew、PS 收集器 复制算法：用在那种不可达对象比较多的场合  


### - cms垃圾收集器的回收步骤及其优缺点？  

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/cms%E5%9B%9E%E6%94%B6.jpg "cms")  


cms回收器的标记过程:

CMS以获取最短回收停顿时间为目标的收集器,使用“标记-清除”算法,分为以下6个步骤

1.STW initial mark:第一次暂停,初始化标记,从root标记old space存活对象(the set of objects reachable from roots (application code))

2.Concurrent marking:运行时标记,从上一步得到的集合出发,遍历old space,标记存活对象 (all live objects that are transitively reachable from previous set)

3.Concurrent precleaning:并发的标记前一阶段被修改的对象(card table)

4.STW remark:第二次暂停,检查,标记,检查脏页的对象,标记前一阶段被修改的对象 (revisiting any objects that were modified during the concurrent marking phase)

5.Concurrent sweeping:运行过程中清理,扫描old space,释放不可到达对象占用的空间

6.Concurrent reset:此次CMS结束后,重设CMS状态等待下次CMS的触发 

或者4个大步骤:   
1,initial mark 2,concurrent mark 3,remark 4,concurrent sweep

CMS缺点：cpu敏感，浮动垃圾，空间碎片

1.CMS收集器对cpu资源非常敏感，在并发阶段对染不会导致用户线程停顿，但是会因为占用一部分线程导致应用程序变慢，总吞吐量会降低。CMS默认启动的收集线程数=(CPU数量+3)/4，在cpu数比较少的情况下，对性能影响较大。

2.CMS收集器无法处理浮动垃圾，可能会出现“Concurrent Mode Failure”失败而导致另一次Full GC，原因是CMS的并发清除阶段用户线程还是在运行，所以还会有新的垃圾不断产生，这些垃圾CMS只能在下次GC时再清理掉，这部分垃圾被称为“浮动垃圾”。所以CMS不能像其他收集器那样在老年代几乎完全被填满了在开始收集，需要预留一部分空间。JDK1.6中CMS将这个阈值提高到了92%，要是CMS运行期间预留的内存不足，会出现一次“Concurrent Mode Failure”，这是虚拟机启动备用方案，临时启用Serial Old收集器充满新进行老年代垃圾收集，所以这个阈值不宜设置的过高

3.CMS基于标记-清除算法，这意味着垃圾收集结束后会有大量的空间碎片，空间碎片过多会造成老年代有很大空间空余但是无法存放大对象的情况。通过参数UseCMSCompactAtFullCollection(默认开启)开关参数来开启内存碎片的合并整理。



### - G1回收器的特点

G1的出现就是为了替换jdk1.5种出现的CMS,这一点已经在jdk9的时候实现了，jdk9默认使用了G1回收器，移除了所有CMS相关的内容。G1和CMS相比，有几个特点：  

1. G1把内存划分为多个独立的区域Region 
2. G1仍然保留分代思想,保留了新生代和老年代,但他们不再是物理隔离,而是一部分Region的集合
3. G1能够充分利用多CPU、多核环境硬件优势，尽量缩短STW
4. G1整体整体采用标记整理算法,局部是采用复制算法,不会产生内存碎片
5. 控制回收垃圾的时间：这个是G1的优势，可以控制回收垃圾的时间，还可以建立停顿的时间模型，选择一组合适的Regions作为回收目标，达到实时收集的目的。控制G1回收垃圾的时间  -XX:MaxGCPauseMillis=200 （默认200ms）
6. G1跟踪各个Region里面垃圾的价值大小,会维护一个优先列表,每次根据允许的时间来回收价值最大的区域,从而保证在有限事件内高效的收集垃圾
7. 大对象的处理: 在CMS内存中，如果一个对象过大，进入S1、S2区域的时候大于改分配的区域，对象会直接进入老年代。G1处理大对象时会判断对象是否大于一个Region大小的50%，如果大于50%就会横跨多个Region进行存放

### - G1执行步骤

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/G1%E6%94%B6%E9%9B%86.jpg "G1")  

1. 初始标记阶段：暂停应用程序，标记可由根直接引用的对象。
2. 并发标记阶段：与应用程序并发进行，扫描 1 中标记的对象所引用的对象。
3. 最终标记阶段：暂停应用程序，扫描 2 中没有标记的对象。本步骤结束后，堆内所有存活对象都会被标记。
4. 筛选回收（首先对各个Regin的回收价值和成本进行排序，根据用户所期待的GC停顿时间指定回收计划，回收一部分Region）


### - 内存泄漏与内存溢出的区别

- 溢出： OOM，除了 PC 剩下的区域都会发生 OOM，是由于内存不够，或者是代码中错误的分配太多对象导致的。
- 泄漏：是指 内存分配后没有回收，导致内存占有一直增加，最后会导致溢出。


### - 有几种gc fail？

Allocation Failure	新生代没有足够的空间分配对象  Young GC	

GCLocker Initiated GC	如果线程执行在JNI临界区时，刚好需要进行GC，此时GC locker将会阻止GC的发生，同时阻止其他线程进入JNI临界区，直到最后一个线程退出临界区时触发一次GC。	  GCLocker Initiated GC

Promotion Failure	老年代没有足够的连续空间分配给晋升的对象（即使总可用内存足够大） 

Concurrent Mode Failure	CMS GC运行期间，老年代预留的空间不足以分配给新的对象	 

### - 什么情况会触发fullgc？

1.metaspace空间不足

2.Promotion Failure	老年代没有足够的连续空间分配给晋升的对象（即使总可用内存足够大） 

3.Concurrent Mode Failure	CMS GC运行期间，老年代预留的空间不足以分配给新的对象	

4.System.gc

5.jmap -histo:live [pid]

### - 栈内存溢出

对虚拟机栈这个区域规定了两种异常状况：

（1）如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError 异常；   
（2）如果虚拟机栈可以动态扩展（当前大部分的 Java 虚拟机都可动态扩展，只不过 Java 虚拟机规范中也允许固定长度的虚拟机栈），当扩展时无法申请到足够的内存时会抛出 OutOfMemoryError 异常。    
（3）与虚拟机栈一样，本地方法栈区域也会抛出 StackOverflowError 和OutOfMemoryError 异常。


### - 系统内存多大，留给操作系统2G，够吗？
16G内存 虚拟机 推荐配置12G堆， 更大的堆容易引起SWAP，SWAP使用过多会造成宕机

各分区的大小对GC的性能影响很大。如何将各分区调整到合适的大小，分析活跃数据的大小是很好的切入点。

活跃数据的大小是指，应用程序稳定运行时长期存活对象在堆中占用的空间大小，也就是Full GC后堆中老年代占用空间的大小

例如，根据GC日志获得老年代的活跃数据大小为300M，

总堆：1200MB = 300MB × 4 新生代：450MB = 300MB × 1.5 老年代： 750MB = 1200MB - 450MB

https://tech.meituan.com/2017/12/29/jvm-optimize.html
