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

在我们声明`NioEventLoopGroup`对象时，代码入口：`io.netty.channel.nio.NioEventLoopGroup#NioEventLoopGroup(int, java.util.concurrent.Executor)`。代码如下：

```Java
public NioEventLoopGroup(int nThreads, Executor executor) {
    this(nThreads, executor, SelectorProvider.provider());
}
```

接着看`java.nio.channels.spi.SelectorProvider#provider`，这里面有一句：

```Java
provider = sun.nio.ch.DefaultSelectorProvider.create();
```

这个方法在不同平台的JDK上是有不同的实现的，也就是说不同的平台使用的多路复用器是不一样的。

## 12 | TCP粘包-半包Netty全搞定

本节讨论的问题：

![](../../笔记图片/36-Netty 源码剖析与实战/tcp粘包和半包的问题.png)

### 什么是粘包和半包

* 粘包

    粘包是指发送方向接收方发送多个数据包时，接收方接收到的数据包并没有按照发送方发送的界限进行区分，而是将多个数据包的数据连续地接收到一起。这通常发生在应用层发送的数据小于TCP层的MSS（最大报文段长度）时，TCP层可能会将多个应用层数据合并为一个TCP段发送出去。接收方在接收到这个TCP段后，如果没有明确的数据边界，就可能出现粘包现象。

    ![](../../笔记图片/36-Netty 源码剖析与实战/粘包的主要原因.png)

* 半包

    半包是指接收方在接收数据时，一个数据包被分成两部分接收，即数据包没有完整地接收。这可能是由于网络延迟、丢包或其他网络问题导致的。当接收方尝试处理一个不完整的数据包时，就会出现半包问题。

    ![](../../笔记图片/36-Netty 源码剖析与实战/半包的主要原因.png)

    ![](../../笔记图片/36-Netty 源码剖析与实战/从收发角度来看.png)

### 为什么TCP应用中会出现粘包和半包现象

根本原因：TCP是流式协议，消息没有边界。

提示：UDP没有粘包和半包的问题，因为他的消息发送和接收，就像邮递的快递，虽然一次运输多个，但是每个包裹都有边界，一个一个签收。

### 解决粘包和半包的几种办法

推荐封装成帧中的第三种，固定长度字段存储内容的长度信息。

![](../../笔记图片/36-Netty 源码剖析与实战/解决粘包和半包的办法.png)

### Netty对三种常用封帧方式的支持

![](../../笔记图片/36-Netty 源码剖析与实战/常用封帧方式的支持.png)

## 13 | 源码剖析：Netty对处理粘包-半包的支持

这里分析的类是：`io.netty.handler.codec.ByteToMessageDecoder#channelRead`

```Java
try {
    ByteBuf data = (ByteBuf) msg;
    // cumulation数据积累器
    first = cumulation == null;
    if (first) {
        // 如果是第一笔数据，直接放入数据积累器中
        cumulation = data;
    } else {
        // 如果不是第一笔数据，就追加到数据积累器中
        cumulation = cumulator.cumulate(ctx.alloc(), cumulation, data);
    }
    // 进行解码
    callDecode(ctx, cumulation, out);
}
```

接着是看`io.netty.handler.codec.ByteToMessageDecoder#callDecode`

暂时放置

##  14 | 常用的二次编码的方式

![](../../笔记图片/36-Netty 源码剖析与实战/常用的二次编解码的方式.png)

![](../../笔记图片/36-Netty 源码剖析与实战/为什么需要二次编解码.png)

![一次二次编解码器](../../笔记图片/36-Netty 源码剖析与实战/一次二次编解码器.png)

![](../../笔记图片/36-Netty 源码剖析与实战/是否能够合并一二次编解码.png)

![](../../笔记图片/36-Netty 源码剖析与实战/常用的二次编解码器.png)

![](../../笔记图片/36-Netty 源码剖析与实战/选择编解码器的要点.png)

![选择编解码器的要点-时间](../../笔记图片/36-Netty 源码剖析与实战/选择编解码器的要点-时间.png)

![选择编解码器的要点-可读性](../../笔记图片/36-Netty 源码剖析与实战/选择编解码器的要点-可读性.png)

![](../../笔记图片/36-Netty 源码剖析与实战/msgpack的优势.png)

![protobuf介绍](../../笔记图片/36-Netty 源码剖析与实战/protobuf介绍.png)

![protobuf的使用](../../笔记图片/36-Netty 源码剖析与实战/protobuf的使用.png)

![protobuf的使用2](../../笔记图片/36-Netty 源码剖析与实战/protobuf的使用2.png)

## 15 | 源码解析：Netty对常用编解码器的支持

Netty对常用编解码器的支持在Netty源码项目中的netty-codec子项目中。

暂时放置

## 16 | keepalive与idle检测

![](../../笔记图片/36-Netty 源码剖析与实战/keepalive与idl监测.png)

![为什么需要keepalive](../../笔记图片/36-Netty 源码剖析与实战/为什么需要keepalive.png)

![](../../笔记图片/36-Netty 源码剖析与实战/为什么需要keepalive2.png)

![怎么设计keepalive](../../笔记图片/36-Netty 源码剖析与实战/怎么设计keepalive.png)

![](../../笔记图片/36-Netty 源码剖析与实战/为什么还需要应用层的keepalive.png)

![为什么还需要应用层的keepalive2](../../笔记图片/36-Netty 源码剖析与实战/为什么还需要应用层的keepalive2.png)

![](../../笔记图片/36-Netty 源码剖析与实战/idle检测是什么.png)

![](../../笔记图片/36-Netty 源码剖析与实战/idle检测是什么2.png)



![](../../笔记图片/36-Netty 源码剖析与实战/如何在Netty中开启空闲检测.png)![](../../笔记图片/36-Netty 源码剖析与实战/如何在Netty中开启空闲检测2.png)

## 17 | 源码剖析：Netty对keepalive与idle检测的支持

![](../../笔记图片/36-Netty 源码剖析与实战/源码解读Netty对TCP keepalive和三种idle检测的支持.png)

暂时放置

## 18 | Netty的那些”锁“事

![](../../笔记图片/36-Netty 源码剖析与实战/Netty的那些锁事.png)

![](../../笔记图片/36-Netty 源码剖析与实战/分析同步问题的三要素.png)

* 原子性：比如i++操作，就涉及到三步：一读取i，二对i进行加一，第三步将和赋值给i，这三步在多线程下执行不一定是连贯的，可能会被别的线程抢先执行。
* 可见性：A线程写入主内存的数据，B线程不一定能看到。
* 有序性：可能会被指令重排导致程序执行的顺序发生变化。

![](../../笔记图片/36-Netty 源码剖析与实战/锁的分类.png)

减少锁的粒度

![](../../笔记图片/36-Netty 源码剖析与实战/锁的对象和范围.png)

减少空间占用

![](../../笔记图片/36-Netty 源码剖析与实战/锁对象本身的大小.png)

注意锁的速度

![](../../笔记图片/36-Netty 源码剖析与实战/注意锁的速度.png)

![](../../笔记图片/36-Netty 源码剖析与实战/注意锁的速度2.png)

不同场景选择不同的并发包

![](../../笔记图片/36-Netty 源码剖析与实战/不同场景选择不同的并发包.png)

![](../../笔记图片/36-Netty 源码剖析与实战/不同场景选择不同的并发包2.png)

衡量好锁的价值

![](../../笔记图片/36-Netty 源码剖析与实战/衡量好锁的价值.png)

 ![](../../笔记图片/36-Netty 源码剖析与实战/衡量好锁的价值2.png)

![衡量好锁的价值3](../../笔记图片/36-Netty 源码剖析与实战/衡量好锁的价值3.png)

![衡量好锁的价值4](../../笔记图片/36-Netty 源码剖析与实战/衡量好锁的价值4.png)

![](../../笔记图片/36-Netty 源码剖析与实战/衡量好锁的价值5.png)

## 19 | Netty玩转内存使用

![](../../笔记图片/36-Netty 源码剖析与实战/内存使用技巧的目标.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-不使用包装类型.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-减少对象本身大小.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-减少对象本身大小2.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-对分配内存进行预估.png)

在初始化的时候就确定容量，方便在后续添加元素的时候进行扩容，从而提升了性能。

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-预测分配大小.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-ZeroCopy.png)

![Netty内存使用技巧-ZeroCopy2](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-ZeroCopy2.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-ZeroCopy3.png)

![Netty内存使用技巧-堆外内存](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-堆外内存.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-堆外内存2.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-内存池.png)![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-内存池2.png)

![](../../笔记图片/36-Netty 源码剖析与实战/Netty内存使用技巧-内存池3.png)

## 20 | 源码解析：Netty对堆外内存和内存池的支持

![](../../笔记图片/36-Netty 源码剖析与实战/源码解读netty内存使用.png)

暂时放置
