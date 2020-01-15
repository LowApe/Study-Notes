# NIO 入门

本章内容：

- 传统的同步阻塞式 I/O 编程

	



## 传统的 BIO

网络编程的基本模型是 **Client/Server**模型，也就是两个进程之间进行相互通信，其中

- 服务端提供位置信息(绑定的 IP 地址和监听端口)
- 客户端通过连接操作向服务端监听的地址发起连接请求，通过三次握手建立连接，如果连接建立成功，双方就可以通过**网络套接字(Socket)**进行通信

下面通过经典的时间服务器 (TimeServer) 为例，通过代码分析来回顾和熟悉下 BIO 编程

### BIO 通信模型图

> 采用 BIO 通信模型的服务端，通常由一个独立的 Acceptor 线程负责**监听客户端的连接**，它接受到客户端连接请求之后，为每个客户端创建一个**新的线程**进行链路处理，处理完成之后，通过输出流返回应答给客户端,**线程销毁**。
>
> 这是典型的一请求一应答通信模型

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1gawyio6fiij31860gm4g1.jpg)

该模型最大的问题就是**缺乏弹性伸缩能力**，当客户端并发访问量增加后，服务端的线程个数和客户端并发访问数呈1:1的正比关系，当线程数膨胀后，系统的性能急剧下降，随着并发访问量的继续增大，系统会发生**线程堆栈溢出**、**创建新线程失败**等问题，并最终导致进程宕机或者僵死，不能对外提供服务



### 同步阻塞式 I/O 创建的 TimeServer 源码分析

```java
public class TimeServer {
    public static void main(String[] args) throws IOException {
        int port = 8080;//服务器监听端口
        if(args != null && args.length > 0){
            try{
                port = Integer.valueOf(args[0]);
            }catch (NumberFormatException e){
				//默认
            }
        }
        ServerSocket serverSocket = null;
        try {
            serverSocket = new ServerSocket(port);
            System.out.println("当前服务器的端口是:" + port);
            Socket socket = null;
            while (true){
                socket = serverSocket.accept();//阻塞，等待客户端请求
                new Thread(new TimeServerHandler(socket)).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(serverSocket!=null){
                System.out.println("The time server close");
                serverSocket.close();
                serverSocket = null;
            }
        }

    }
}
```

TimeServer 根据传入的参数设置监听端口，如果没有入参，使用默认值 8080，服务端监听成功;之后通过一个无限循环来监听客户端的连接，如果没有客户端接入，则主线程阻塞在 ServerSocket 的 accept 操作上



### 同步阻塞 I/O 的TimeServerHandler

```java
public class TimeServerHandler implements Runnable{
    private Socket socket;
    public TimeServerHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        BufferedReader in = null;
        PrintWriter out = null;
        try {
            in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
            out = new PrintWriter(this.socket.getOutputStream(),true);
            String currentTime = null; //当前时间
            String body = null;//接受客户端内容
            while(true){
                body = in.readLine();//读取 输入过来的信息
                if(body == null){
                    break;
                }
                System.out.println("接收到的信息:" + body);
                currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new Date(System.currentTimeMillis()).toString() : "BAD QUERY";
                out.println("当前时间是:"+currentTime);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

通过 BufferedRead 读取一行，

- 如果已经读到了输入流的尾部,则返回值为 null,退出循环。
- 如果读到了非空值，则对内容进行判断，如果请求消息为查询时间的指令 “QUERY TIME ORDER” 则获取当前最新的系统时间，通过 **PrintWriter 的 println 函数发送给客户端**，最后退出循环。
- 释放资源

### 同步阻塞式 I/O 创建的 TimeClient 源码分析

客户端通过 Socket 创建，发送查询时间服务器的 “QUERY TIME ORDER” 指令，然后读取服务器的响应并将结果打印出来，随后关闭连接，释放资源，程序退出执行。

```java
public class TimeClient {
    public static void main(String[] args) {
        int port = 8080;
        if (args != null && args.length > 0) {
            try {
                port = Integer.valueOf(args[0]);
            } catch (NumberFormatException e) {
                //默认
            }
        }

        BufferedReader in = null;
        PrintWriter out = null;
        Socket socket = null;
        try {
            socket = new Socket("127.0.0.1", port);
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream(), true);
            out.println("QUERY TIME ORDER");
            System.out.println("发送请求成功");
            String resp = in.readLine();
            System.out.println("响应信息:"+resp);
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(out!=null){
                out.close();
                out = null;
            }

            if(in != null){
                try {
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                in = null;
            }
            if(socket != null){
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                socket = null;
            }

        }
    }
}
```

### 总结

BIO 模型每当一个新的客户端请求，服务端必须创建一个新的线程处理新接入的客户端链路，一个线程只能处理一个客户端连接。

在高性能服务器应用领域，往往需要向成千上万个客户端的并发连接，这种模型无法满足高性能、高并发接入的场景

后来又演进出了一种通过**线程池或者消息队列**实现 1 个或者多个线程处理 N 个客户端的模型，因为底层通信机制依然使用同步阻塞 I/O，所以被称为“伪异步”



## 伪异步 I/O 编程

