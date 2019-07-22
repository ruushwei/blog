---
title: netty模型
date: 2019-07-09T12:54:24+02:00
tags: 
- netty
categories: netty
draft: false
---

#### 定义

netty是一个高性能、异步事情驱动的网络通信框架  
NIO IO多路复用 、事情驱动模型

1.设置主从reactor模式  
2.指定IO类型  
3.指定handler
4.绑定端口

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/640.jpeg)


#### netty优点

1. API使用简单，开发门槛低；  

2. 功能强大，预置了多种编解码功能，支持多种主流协议；  

3. 定制能力强，可以通过ChannelHandler对通信框架进行灵活地扩展；  

4. 性能高，通过与其他业界主流的NIO框架对比，Netty的综合性能最优；  

5. 成熟、稳定，Netty修复了已经发现的所有JDK NIO BUG，业务开发人员不需要再为NIO的BUG而烦恼；  

6. 社区活跃，版本迭代周期短，发现的BUG可以被及时修复，同时，更多的新功能会加入；  

7. 经历了大规模的商业应用考验，质量得到验证。在互联网、大数据、网络游戏、企业应用、电信软件等众多行业得到成功商用，证明了它已经完全能够满足不同行业的商业应用了。

#### 高性能三要素

1. reactor模型
2. IO多路复用
3. 协议

#### reactor模型

##### 单线程模型

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/单.jpg)

##### 多线程模型

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/多.jpg)

##### 主从线程模型

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/主从.jpg)

#### 组件

##### Bootstrap/ServerBootstrap  

顾名思义就是启动类，分别负责启动客户端和服务器端。这个类用来配置相关参数，比如设置EventLoopGroup，IO类型，handler等等，Netty的一切都从这里开始

##### EventLoopGroup

主从多线程Reactor模型，分别有两个线程池：  EventLoopGroupA和EventLoopGroupB。一个用于接收请求，另一个用于处理IO操作。

一个EventLoopGroup就相当于一个线程池，而每一个EventLoop就是一个线程，当新的Channel被创建时（有新的请求进来），就会在EventLoopGroup里注册一下，同时会分配一个EventLoop给这个Channel，

从此开始直到这个Channel被销毁，这个Channel只能被它绑定的这个EventLoop执行，这也就是为什么Netty可以不用考虑并发的原因。

EventLoop是处理各个event的具体线程。除了处理IO读写等event外，EventLoop还需要进行系统任务和定时任务进行执行

##### Channel
Netty的Channel接口所提供的的API，大大降低了直接使用Socket类的复杂性

##### ChannelPipeline & ChannelHander  

Netty采用了一种叫做数据流（data flow）的处理机制，类似于Unix中的管道。即每一个Channel都有一个自己的ChannelPipeline，每一个pipeline里会有多个ChannelHandler。数据会像水流一样依次通过每一个handler被逐一处理。  
流处理是双向混合的，分为Inbound和Outbound， 分别对应request和response。

这个handler被分成两类：ChannelOutboundHandler和ChannelInboundHandler。当服务器处理进来的请求时，则只会调用实现了ChannelInboundHandler的handler；当服务器返回信息给客户端时，则只会调用实现了ChannelOutboundHandler的handler

##### Encoders & Decoders

我们在解析处理请求时通常需要对数据格式进行转换，比如把字节变成对象，或者把对象转换为字节。针对这种常见的场景，Netty提供了编码和解码的接口：MessageToByteEncoder和ByteToMessageEncoder。

其实两个抽象类分别继承了ChannelInboundHandlerAdapter和ChannelOutboundHandlerAdapter，说白了，使用起来和普通的handler没什么区别。自己写的类只要重写decode()或者encode()方法对数据进行处理即可
