# 第二章

## 08 Netty怎么切换三种IO模式

![](../../笔记图片/36-Netty 源码剖析与实战/Netty怎么切换三种io模式.png)

![什么是经典的三种io模式](../../笔记图片/36-Netty 源码剖析与实战/什么是经典的三种io模式.png)

![io模式类比生活场景](../../笔记图片/36-Netty 源码剖析与实战/io模式类比生活场景.png)

![](../../笔记图片/36-Netty 源码剖析与实战/阻塞和同步.png)

 ![](../../笔记图片/36-Netty 源码剖析与实战/Netty对三种IO的支持.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty为什么只支持NIO.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty中为什么NIO会有多种实现.png)

![](../../笔记图片/36-Netty 源码剖析与实战/为什么要自己实现其他系统的实现.png)

![](../../笔记图片/36-Netty 源码剖析与实战/BIO一定优于NIO吗.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty是如何切换IO模式的.png)

## 09 | 源码剖析：Netty对IO模式的支持

这一节主要介绍了上一节最后一张片子的三个问题，下面我来一一总结。

* 怎么切换
    1. 在服务端主程序中，将`NioEventLoopGroup`切换为`AIOEventLoopGroup`或是`OioEventLoopGroup`就可以切换为BIO或是AIO。
    1. 将`channel(NioServerSocketChannel.class)`中的`NioServerSocketChannel`换成`AIOServerSocketChannel`或是OioServerSocketChannel。
    
* 原理是什么

    * 在`NioEventLoop`中你就可以看到一个`run()`方法，这个是Netty实现个Reactor模型的关键。
    * 在`io.netty.bootstrap.AbstractBootstrap#channel`方法中，是通过**反射+泛型+工厂模式**实现了IO的切换。

* 为什么服务器开发不需要像客户端一样切换为Socket对象

    因为`NioServerSocketChannel`中有一个`o.netty.channel.socket.nio.NioServerSocketChannel#doReadMessages`方法，在这个方法中底层是通过Java的`ServerSocketChannel`封装的。

## 10 | Netty如何支持三种Reactor

![](../../笔记图片/36-Netty 源码剖析与实战/什么事Reactor及三种版本.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Reactor生活场景.png)

我们把服务端处理请求的过程抽象成我们饭店的生活场景，上面的第一种情况就是单线程，第二种是多线程，第三种是主从多线程，其中迎宾是饭店最重要的事情。

![](../../笔记图片/36-Netty 源码剖析与实战/Reactor生活场景类比.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Reactor对照三种io模型.png)

上图中针对Server端，`ServerSocketChannel`最重要的事情是接收客户端的请求，也就是”迎宾“。

![](../../笔记图片/36-Netty 源码剖析与实战/Thread-Per-Connection模式.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Thread-Per-Connection模式的代码实现.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Reactor模式V1-单线程.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Reactor模式V2-多线程.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Reactor模式V3-主从多线程.png)

![](../../笔记图片/36-Netty 源码剖析与实战/如何在Netty中使用Reactor的三种模式.png)

## 11 | 源码剖析：Netty对Reactor的支持

Netty对Reactor模式支持的常见疑问：

![](../../笔记图片/36-Netty 源码剖析与实战/解析Netty对Reactor模式支持的常见疑问.png)

### 第一个问题：Netty是如何支持主从Reactor模式的？

这个方法的入口是：`ServerBootstrap#group(io.netty.channel.EventLoopGroup, io.netty.channel.EventLoopGroup)`，在这里会将bossGroup赋值给`io.netty.bootstrap.AbstractBootstrap#group`对象。这个对象会在下面bind中的`io.netty.bootstrap.AbstractBootstrap#initAndRegister`方法中被使用，也就是下面的这句：

```Java
ChannelFuture regFuture = config().group().register(channel);
```

这里的`config().group()`就是bossGroup，channel就是`NioServerSocketChannel`，对应与Java NIO中的`ServerSocketChannel`。

这个方法是的调用入口是：`ServerBootstrap.bind()`

![](../../笔记图片/36-Netty 源码剖析与实战/将Netty的Channel注册到Selector.png)

最后，这个NioServerSocketChannel最后注册到Selector选择器上，其中在注册选择器的时候，会从bossGroup的线程组中选一个线程去做这个注册工作。

接着我们来看workGroup，在`io.netty.bootstrap.ServerBootstrap.ServerBootstrapAcceptor#channelRead`方法中。方法的参数是`msg`，是一个`NioSocketChannel`，当客户端发起请求的时候就会走到这里，将客户端请求的channel注册到workGroup上。

综上所述，bossGroup负责accept请求，workGroup负责请求的读操作和写操作。

### 第二个问题：为什么说Netty的main reactor只会用到线程池中的一个线程？

代码的关键入口是：`io.netty.util.concurrent.MultithreadEventExecutorGroup#next`，调用的对账信息如下图：

![](../../笔记图片/36-Netty 源码剖析与实战/mainReactor只会使用一个线程.png)

这里的这个`chooser`的实现有两种，一个是`PowerOfTowEventExecutorChooser`，一个是`GenericEventExecutorChooser`，使用这两种的哪一种，代码如下：

```Java
public EventExecutorChooser newChooser(EventExecutor[] executors) {
    // 判断bossGroup的线程池数量是否为 2 的幂次方
    if (isPowerOfTwo(executors.length)) {
        return new PowerOfTowEventExecutorChooser(executors);
    } else {
        return new GenericEventExecutorChooser(executors);
    }
}
```

这两种具体的next方法，其实就是取线程池中的一个线程去处理main Reactor的逻辑。在上面的堆栈图中我们看到，最开始触发的逻辑的`ServerBootstrap.bind()`方法，对于服务器端来说，它的启动只会绑定一个端口，所以只有一个线程。

### 第三个问题：Netty给NioSocketChannel分配NioEventLoop的规则是什么？

`NioEventLoop`代表一个处理线程。bossGroup和workGroup如果不指定线程数的话，都会创建`CPU核心数*2`的`NioEventLoop`。具体的分配规则就是上面说到的`PowerOfTowEventExecutorChooser`和`GenericEventExecutorChooser`。方法的入口在：`io.netty.util.concurrent.MultithreadEventExecutorGroup#next`。

### 第四个问题：通用模式的NIO实现多路复用器是怎么跨平台的





