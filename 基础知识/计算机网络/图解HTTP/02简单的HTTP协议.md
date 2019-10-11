

# 简单的HTTP协议

本章内容

- 针对HTTP协议结构进行讲解。HTTP/1.1版本

## HTTP协议用于客户端和服务端之间的通信

- 客户端：请求访问文本或图像等资源的一端
- 服务端：提供资源响应的一端

## 通过请求和响应的交换达成通信

首先从客户端开始**建立通信**，服务端在没有接受到请求之前不会发送响应

|      | 客户端                                                       | 服务端                                                       |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 报文 | GET /index.htm HTTP/1.1<br />Host:hackr.jp<br />请求报文=请求方法+请求URI+协议版本+可选的请求首部字段+内容实体 | HTTP/1.1 200 OK<br />Date:Tue,10 May 2020 12:12:12 GMT<br />Content-Length:362<br />Content-Type:text/html<br />响应报文=协议版本+状态码+解释状态码原因短语+创建响应时间+首部字段+空一行分隔+资源实体的主体 |

## HTTP是不保存状态的协议

HTTP是无状态的协议。HTTP协议自身不对请求和响应之间的通信进行保存

<img src="http://ww1.sinaimg.cn/large/006rAlqhly1g7ucoqgpvvj30ps094445.jpg" alt="image.png" style="zoom:50%;" />

> 使用HTTP协议，每当有新的请求发送时，就会有对应的新响应产生。协议本身并不保留之前一切的请求或响应报文的信息。这是为了更快地处理大量事务，确保协议的可伸缩性，而特意把HTTP协议设计成如此简单的

HTTP/1.1虽然无状态协议，但为了实现期望的保持状态功能。引入了**Cookie技术**。有了Cookie再用HTTP协议通信，就可以管理状态了。

## 请求URI定位资源

HTTP协议使用URI定位互联网上的资源。正是因为URI的特定功能，在互联网上任意资源的资源都能访问到。

当客户端请求访问资源而发送请求时，URI需要将作为请求报文中的请求URI包含在内。指定请求URI的方式有很多

- GET http://hackr.jp/index.htm HTTP/1.1
- GET /index.htm HTTP/1.1
    Host:hackr.jp--服务器地址

> ⚠️提示：如果不是访问特定资源而是对服务器本身发起请求，可以用一个 `*` 来代替请求 URI。下面这个例子是查询HTTP服务器端支持的HTTP方法种类 `OPTIONS * HTTP/1.1`

## 告知服务器意图的HTTP方法

| HTTP方法        | 描述                                                         |                                                              |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| GET             | GET方法用来请求访问已被URI识别的资源                         | 请求访问资源                                                 |
| POST            | POST方法用来**传输实体**的主体                               | 向服务端发送参数数据                                         |
| PUT             | PUT方法用来**传输文件**                                      | 要求请求报文中包含文件内容，然后保存到请求 URI 指定的位置<br />鉴于HTTP/1.1的PUT方法自身不带验证机制，任何人都可以上传文件，存在安全性问题<br />一般配合Web应用程序的验证机制，或架构设计采用REST（REpresentational State Transfer，表征状态转移） |
| HEAD            | HEAD 方法和GET方法一样，只是不返回报文主体部分               | 用于确认URI的有效性及资源更新的日期时间等                    |
| DELETE          | DELETE方法用来删除文件，是与PUT相反的方法                    | DELETE方法按请求URI删除指定的资源                            |
| OPTIONS         | OPTIONS方法用来查询针对请求URI指定的资源支持的方法           | 你支持哪些方法？支持 GET 和 HEAD 方法                        |
| TRACE(追踪路径) | TRACE 方法是让 Web服务器端将之前的请求通信**环回**给客户端的方法 | 客户端通过 TRAC 方法可以查询发送出去的请求是怎样被加工修改/篡改的<br />TRACE 方法本来就不怎么常用，加上它容易引发 XST（Cross-Site Tracing，跨站追踪）攻击 |
| CONNECT         | CONNECT 方法要求在与**代理服务器**通信时建立隧道，实现用隧道协议进行TCP通信 | 主要使用SSL(Secure Socket Layer,安全套接层)和TLS(Transport Layer Security,传输层安全)<br />`CONNECT代理服务器名：端口号 HTTP版本 ` |

trace：

![image-20191011175457269](/Users/mac/Library/Application Support/typora-user-images/image-20191011175457269.png)

connect

## 使用方法下达命令

向请求 URI 指定的资源发送请求报文时，采用称为方法的命令。

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7uf4sdmj4j30m80ckade.jpg)

## 持久连接节省通信量

HTTP协议的初始版本中，每进行一次HTTP通信就要断开一次TCP连接，通信初期传输都是容量很小的文本传输，现在传输的内容类型很多，每次的请求都会造成无谓的TCP连接建立和断开，增加通信量的开销

### 持久连接

为解决上述TCP连接的问题，HTTP/1.1和一部分的HTTP/1.0 想出了**持久连接（HTTP keep-alive）**的方法。持久连接的特点是，只要**任意一端没有明确提出断开连接**，则保持TCP连接状态。

### 管线化

持久连接使得多数请求以**管线化(pipelining)**方式发送称为可能。从前发送请求后需等待并收到响应，才能发送下一条请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求

## 使用 Cookie 的状态管理

无状态优势是减少服务器的CPU及内存资源的消耗。从另一侧面来说，也正是因为HTTP本身简单，所以才被应用到各种场景里。

> 让服务器管理全部客户端状态则会成为负担

Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie的首部字段信息，通知客户端保存 Cookie。下次客户端再往服务器发送请求时，客户端会自动在请求报文中加入Cookie值后发送出去。

HTTP请求报文和响应报文的内容如下：

1. 请求报文

```http
GET /reader/ HTTP/1.1
Host:hackr.jp
* 首部字段内没有Cookie 的相关信息
```

2. 响应报文（服务器端生成 Cookie 信息的状态）

```http
HTTP/1.1 200 OK
Date: Thu, 12 Jul 2020 12:12:12 GMT
Server: Apache
<Set-Cookie:JSESSIONID=239702CC83BEF313BB76A490A5B8A47C;Path=/;expires =Wed,10-Oct-12 07:12:12 GMT>
```

3. 请求报文（自动发送保存着的Cookie信息）

```http
GET /image/ HTTP/1.1
Host: hackr.jp
Cookie: sid=xzczxczczxczcx
```

