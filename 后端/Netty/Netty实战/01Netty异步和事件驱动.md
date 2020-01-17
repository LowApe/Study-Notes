# Netty 异步和事件驱动

主要内容:

- Java 网络编程
- Netty 简介
- Netty 的核心组件

> Netty 是一款异步的事件驱动的网络应用程序框架，支持快速地开发可维护的高性能的面向协议的服务器和客户端

## Java 网络编程

早前的 Java API(java.net) 只支持由**本地系统套接字库提供的所谓的阻塞函数**

```java
// 创建一个新的ServerSocket，用以监听指定端口上的连接请求
ServerSocket serverSocket = new ServerSocket(portNumber);
Socket clientSocket = serverSocket.accept();
BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
// 这些流对象都派生于该套接字的流对象
PrintWriter out = new PrintWriter(clietnSocket.getOutputStream(),true);
String request,response;
while((request = in.readLine()) != null){
    if("Done".equals(request)){
        break;//如果客户端发送了 “Done”,则退出处理循环
    }
    response = processRequest(request);//请求被传递给服务器的处理方法
    out.println(response);//服务器的响应被发送给了客户端
}
```

- ServerSocket 上的 accept() 方法将会一直阻塞到一个连接创建，随后返回一个新的 Socket 用于客户端和服务器之间的通信
- BufferedReader 和 PrintWriter 都衍生自 Socket 的输入输出流。前者从一个字符输入流中读取文本，后者打印对象的格式化的表示到文本输出流
- readLine() 方法将会阻塞，直到在处一个由换行符或者返回结尾的字符串被读取
- 客户端请求被处理

这种方案只能同时处理一个连接，要管理多个并发客户端，需要为每个新的客户端 Socket 创建一个新的 Thread。在高并发的情况下会存在栈溢出，连接失败，等由于系统资源无法处理

## Java NIO

新的是非阻塞的(Non-blocking I/O),而阻塞 I/O 是旧的输入/输出

## 选择器

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1gaxdxy0ug7j30x80qgn0c.jpg)



class `java.nio.channels.Selector`是 Java的非阻塞 I/O 实现的关键。他使用了**事件通知 API** 以确定在一组非阻塞套接字中有哪些已经就绪能够进行 I/O 相关的操作。因为可以在任何的时间检查任意的读操作或者写操作的完成状态。

这种模型更好的资源管理：

- 使用较少的线程便可以处理许多连接，因此也减少了内存管理和上下文切换所带来开销;
- 当没有 I/O 操作需要处理的时候，线程也可以被用于其他任务

## Netty简介

> 在网络编程领域，Netty 是 Java 的卓越框架。它架奴了 Java 高级 API 的能力，并将其隐藏在一个易于使用的 API 之后。

| 分类     | Netty的特性                                                  |
| -------- | ------------------------------------------------------------ |
| 设计     | 统一的API，支持多种传输类型，阻塞的和非阻塞的<br />简单而强大的线程模型<br />真正的无连接数据报套接字支持<br />链接逻辑组件以支持复用 |
| 易于使用 | 详实的 Javadoc 和大量的示例集<br />不需要超过 JDK1.6+的依赖  |
| 性能     | 拥有比Java的核心API 更高的吞吐量以及更低的延迟<br />得益于池化和复用，拥有更低的资源消耗<br />最少的内存复制 |
| 健壮性   | 不会因为慢速、快速或者超载的连接而导致 OutOfMemoryError<br />消除在高速网络中 NIO 应用程序常见的不公平读/写比率 |
| 安全性   | 完整的 SSL/TLS 以及 StartTLS 支持可用于受限环境下，如 Applet 和 OSGI |
| 社区驱动 | 发布快速而且而且频繁                                         |

### 异步和事件驱动

一个异步和事件驱动的系统它可以以任意的顺序响应在任意的时间点产生的事件

> 这种能力对于实现最高级别的可伸缩性至关重要，定义为:"一种系统、网络或者进程在需要处理的工作不断增长时，可以通过某种可行的方式或者扩大它的处理能力来适应这种增长的能力"

异步和可伸缩性之间的联系是什么？

- 非阻塞网络调用使得我们可以不必等待一个操作的完成。完全异步的 I/O 正是基于这个特性构建的，并且更进一步：异步方法会立即返回，并且在它完成时，会直接或者在稍后的某个时间点通知用户
- 选择器使我们能够通过较少的线程便可监视许多连接上的事件



## Netty的核心组件

Netty 的主要**构件块**：

- Channel
- 回调
- Future
- 事件和 ChannelHandler

> 这些构件块代表了不同类型的构造:
> 资源
> 逻辑
> 通知



### Channel

Channel 是 Java NIO 的一个基本构造

> 它代表一个到实体(如一个硬件设备、一个文件、一个网络套接字或者一个能够执行一个或者多个不同的 I/O 操作的程序组件)的开发连接，如读操作和写操作

目前，可以把 Channel 看作是传入 (入站) 或者传出 (出战) 数据的载体。因此，它可以被打开或者被关闭，连接或者断开连接。

### 回调

Netty 在内部使用了回调来处理事件;当一个回调被触发时，相关的事件可以被一个 interface-ChannelHandler 的实现处理。当一个新的连接已经被创建时，ChannelHandler 的 channelActive() 回调方法将会被调用，并将打印出一条信息

```java
public class ConnectHandler extends ChannelInboundHandlerAdapter{
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception{//当一个新的连接已经被创建时改方法会被调用
            System.out.println("Client " + ctx.channel().remoteAddress() + " connected");
    }
}
```

### Future 

JDK 预置了 interface `java.util.concurrent.Future`，但是其所提供的实现，只允许手动检查对应的操作是否已经完成，或者一直阻塞直到它完成。由于非常繁琐的，所以 Netty 提供了它自己的实现——**ChannelFuture**,用于在执行异步操作的时候使用

ChannelFuture 提供了几种额外的方法，这些方法使得我们能够注册一个或者多个 ChannelFutureListener 实例。监听器的回调方法 operationComplete() ，将会在对应的**操作完成时被调用**。然后监听器可以判断该操作是成功地完成了还是出错了。如果是后者，我们可以检索产生的 Throwable 。见而言这，由 ChannelFutureListener 提供的通知机制消除了手动检查对应的操作是否完成的必要

每个 Netty 的出站 I/O 操作都将返回一个 ChannelFuture;也就是说，我们都不会阻塞。正如我们前面所提到过的一样，Netty 完全是异步和事件驱动的。



异步地创建连接

```java
Channel channel = ...;
//异步地连接到远程节点
ChannelFuture future = channel.connect(
	new InetSocketAddress("192.168.0.1",25));
```

要注册一个新的 ChannelFutureListener 到对 connect() 方法的调用所返回的 ChannelFuture 上。当该监听器被通知连接已经创建的时候



回调实战

```java
Channel channel = ...;
ChannelFuture future = channel.connect(
	new InetSocketAddress("192.168.0.1",25) 
);
//注册一个 ChannelFutureListener，以便在操作完成时获得通知
future.addListener(new ChannelFutureListener(){
    @Override
    public void operationComplete(ChannelFuture future){//检查操作的状态
        if(future.isSuccess()){
            //如果操作是成功的，则创建一个 ByteBuf 以持有数据
            ByteBuf buffer = Unpooled.copiedBuffer(
            	"Hello",Charset.defaultCharset());
            
            //将数据异步地发送到远程节点
            ChannelFuture wf = future.channel()
                .writeAndFlush(buffer);
        } else {
            //如果发生错误，则访问描述原因的 Throwbale
            Throwable cause = future.cause();
            cause.printStackTrace();
        }
    }
})
```

如果你把 ChannelFutureListener 看作是回调的个更加精细的版本，那么你是对的。事实上，回调和 Future 是相互补充的机制；它们相互结合，构成了 Netty 本身的关键构件块之一。

### 事件和 ChannelHandler

当我们状态的改变或者是操作的状态改变，Netty 使用不同的事件来通知。在基于发生的事件来触发适当的动作。这些动作可能是:

- 记录日志；
- 数据转换；
- 流控制；
- 应用程序逻辑;

Netty 是一个网络编程框架，所以事件是按照它们与入站或出站数据流的相关性进行分类的。可能由**入站数据**或者相关联的状态更改而触发的事件包括:

- 连接已被激活或者连接失活；
- 数据读取;
- 用户事件;
- 错误事件;

出站事件是未来将会触发的某个动作的操作结果，这些动作包括:

- 打开或者关闭到远程节点的连接;
- 将数据写到或者冲刷到套接字;

每个事件都可以被分发给 ChannelHandler 类中的某个用户实现的方法。这是一个很好的将**事件驱动范式直接转换为应用程序构件块**的例子。

![image-20200117094158641](/Users/mac/Library/Application Support/typora-user-images/image-20200117094158641.png)

Netty 的 ChannelHandler 为处理器提供了基本的抽象。我们会在适当的时候对 ChannelHandler 进行更多的说明，但是目前你可能认为每个 Channel-Handler 的实例都类似于一种为了响应特定事件而被执行的回调。

Netty 提供了大量预定义的可以开箱即用的 ChannelHandler 实现，包括用于各种协议(如 HTTP 和 SSL/TLS ) 的 ChannelHandler 。在内部，ChannelHandler 自己也使用了事件和 Future,使得它们也成为了你的应用程序将使用的相同抽象的消费者

### 把他们放在一起

介绍了 Netty 实现高性能网络编程的方式，以及它的实现中的一些主要的组件。让我们大体回顾一下我们讨论过的内容吧。

1. **Future、回调和 ChannelHandler**

Netty 的异步编程模型是创建在 **Future 和回调**的概念之上的，而将事件派发到 ChannelHandler 的方法则发生在更深的层次上。结合在一起，这些元素就提供了一个处理环境，使你的应用程序逻辑可以独立于任何网络操作相关的顾虑而独立地演变。这也是 Netty 的设计方式的一个关键目标。

拦截操作以及高速地转换入站数据和出站数据，都只需要你提供回调或者利用操作所返回的 Future。这使得链接操作变得既简单又高效，并且促进了可重用的通用代码的编写

2. **选择器、事件和 EventLoop**

	Netty 通过触发事件将 Selector 从应用程序中抽象出来，消除了所以本来将需要手动编写的派发代码。在内部，将会为每个 Channel 分配一个 EventLoop，用以处理所有事件，包括：

	- 注册感兴趣的事件;
	- 将事件派发给 ChannelHandler
	- 安排进一步的动作

EventLoop 本身只由一个线程驱动，其处理了一个 Channel 的所有 I/O 事件，并且在该 EventLoop 的整个生命周期内都不会改变。这个简单而强大的设计消除了你可能有的在 ChannelHandler 实现中需要进行同步的任何顾虑，因此，你可以专注于提供正确的逻辑，用来在有感兴趣的数据要处理的时候执行。如同我们在详细探讨 Netty 的线程模型时将会看到的，该 API 是简单而紧凑的

## 小结

- Netty 框架的背景知识
- Java 网络编程 API 的演变过程
- 阻塞和非阻塞网络操作之间的区别
- 异步 I/O 在高容量、高性能的网络编程中的优势。
- Netty 的特性、设计和优点

