---
title: netty服务端启动原理
date: 2021-08-23 11:10:43
tags: 
- netty
categories: netty
---

Netty 是一个事件驱动模式的高性能的 NIO 框架，对内封装了 Java NIO 的大量复杂且繁琐的实现细节，对外提供了高度抽象的组件和易用的 API，提供了针对市面上大部分通信协议的支持，是当下 Java 生态中最流行的远程通信框架之一。

**Netty 框架本质上是对于 Java NIO 的封装，Java NIO 本质上是对于操作系统提供的 IO 多路复用功能的封装**，因此其设计思路是一脉相承的，即采取了事件驱动的 Reactor 模式。

其中的 bossGroup 就是 mainReactor 的具体实现，主要用于监听服务端口，接收客户端的 TCP 连接请求。 workerGroup 就是 subReactor 的具体实现，主要用于消息的读取发送、编解码以及其他业务逻辑的处理。

它们本质上都可以视作一个线程池，里面的每个线程（NioEventLoop）都唯一绑定了一个 Java NIO 中的 Selector 对象，用于实现 IO 多路复用的功能，来监听多个（文件描述符）客户端的输入。


![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/netty/netty%E6%9E%B6%E6%9E%84.png)


启动demo
```java
public class DiscardServer {

    private int port;

    public DiscardServer(int port) {
        this.port = port;
    }

    public void run() throws Exception {
          // mainReactor(Acceptor)线程池，主要用于监听服务端口，处理客户端连接
        EventLoopGroup bossGroup = new NioEventLoopGroup();
          // subReactor线程池，主要用于实现对于消息的接收发送、编解码以及其他业务处理
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            // Netty的启动辅助类
            ServerBootstrap b = new ServerBootstrap();
            // 初始化ServerBootstrap实例
            b.group(bossGroup, workerGroup)
             // 指定Channel类型为NioServerSocketChannel
             .channel(NioServerSocketChannel.class)
             // 添加自定义Handler，会用于workerGroup中的Channel的处理
             .childHandler(new ChannelInitializer<SocketChannel>() {
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             // 添加Channel选项
             .option(ChannelOption.SO_BACKLOG, 128)
             .childOption(ChannelOption.SO_KEEPALIVE, true); 

            // Bind and start to accept incoming connections.
            // 服务端启动核心逻辑，初始化Channel并绑定指定服务端口开始监听
            ChannelFuture f = b.bind(port).sync();

            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            // 服务端关闭
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port = 8080;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        }

        new DiscardServer(port).run();
    }
}
```


1. 初始化： 创建 ServerBootstrap 实例并进行初始化
2. 启动服务： ServerBootstrap 绑定服务端口并开始监听







