# 了解Web及网络基础
本章内容
- HTTP协议如何诞生的

## 使用HTTP协议访问Web
使用场景:当在游览器的地址栏内输入URL时之后，信息会被送到某处，然后从某处获得的回复，内容就会显示在Web页面上。
- 这之间使用 HTTP 协议的通信

Web 使用一种名为 HTTP(HyperText Transfer Protocol,超文本传输`转移`协议)作为规范，完成从客户端到服务器端等一系列**运作流程**。而协议是指规则的约定。可以说，Web是建立在HTTP协议上通信的

## HTTP 的诞生

| 发展历程     | 描述     |
| :------------- | :------------- |
| 为知识共享而规划Web       | 最初设想的基本概念：借助多文档之间相互关联形成的超文本(HyperText)，连成可相互参阅的**WWW**|
|Web成长时代|CERN成功研发第一台服务器与游览器|
|驻足不前的HTTP|当年HTTP协议的出现主要是为了解决文本传输的难题。现在HTTP协议已经超出了Web这个框架的，被运用到了各种场景里|

## 网络基础 TCP/IP

了解HTTP之前，有必要了解一下 TCP/IP协议族。
通常使用的网络是在 TCP/IP 协议族的基础上运作的。而HTTP是属于它内部的子集

### TCP/IP协议族
计算机要与网络设备相互通信，双方就必须基于相同的方法。比如：
- 如何探测到通信目标
- 由哪一边发起通信
- 使用哪种语言进行通信
- 怎样结束通信等规划都需要事先确定

不同的硬件、操作系统之间的通信，所有的这一切都需要一种规则。我们把这种**规则称为协议（protocol）**

> TCP/IP 是互联网相关的**各类协议族**的总成

### TCP/IP的分层管理
TCP/IP最重要的一点是分层，按照层次分为4类

- 应用层
- 传输层
- 网络层
- 数据链路层

TCP/IP 层次化的好处。比如，如果互联网只由一个协议族某个地方需要改动设计时，就必须把整体替换。而分层之后只需要把变动的层替换掉即可。把各层之间的接口部分规划好之后，每层内部的设计就能够自由改动了。

| 分层 |   描述   |提供服务|
| :-------------: | :------------- |:------------- |
|   应用层     |   决定向用户提供应用服务时通信     |`FTP`(File Transfer Protocol)和`DNS`(Domain Name System)服务、`HTTP`|
|    传输层    |   对上层应用层提供处于网络连接中的两台计算机之间的数据传输  |`TCP`(Transmission Control Protocol)和`UDP`(User Data Protocol,用户数据报协议)|
|   网络层     |   用来处理网络上流动的数据包。数据包是网络传输的最小数据单位。   |该层规定了通过怎样的路径(所谓的传输路线)到达对方计算机，并把数据包传送给对方。IP（Internet Protocol）协议  |
|     数据链路层   |  用于处理连接网络的硬件部分      |包括控制操作系统、硬件的设备驱动、`NIC`(Networkd Interface Card，网络适配器，即网卡)，硬件上的范畴都在链路层的作用范围之内|

### TCP/IP通信传输流

<img src="http://ww1.sinaimg.cn/large/006rAlqhly1g7qksb1y0lj30z60r0jvw.jpg" alt="image.png" style="zoom:50%;" />

- 首先作为发送端的客户端在应用层(HTTP协议)发出一个想看某个Web页面的HTTP请求

- 传输层把从应用层处收到的数据(HTTP请求报文)进行**分割**，并在各个**报文打上标记序号**及**端口号**后**转发网络层**

- 网络层，增加作为通信目的地的MAC地址后转发给链路层

接收端的服务器在链路层接受到数据，按序往上层发送，一直到应用层。当传输到应用层，才能算真正接收到由客户端发送过来的HTTP请求

<img src="http://ww1.sinaimg.cn/large/006rAlqhly1g7qlq00c9pj30xa0qw7e3.jpg" alt="image.png" style="zoom:50%;" />

> 这种把数据信息包装起来的做法称为封装

## 与 HTTP 关系密切的协议：IP、TCP、和DNS

下面我们分别针对在 TCP/IP 协议族中与HTTP密不可分的3个协议(IP、TCP和DNS)进行说明

### 负责传输的IP协议

按层次分，IP网际协议位于**网络层**。

> ⚠️注意：可能有人会把“IP”和“IP地址”搞混，“IP”其实是一种协议的名称

IP协议的作用是把各种**数据包传送给对方**。而要**保证确实传送**到对方那里，则需要满足各类条件

- IP地址（指明了节点分配到的地址）
- MAC地址（指**网卡**所属的**固定地址**）

IP间的通信依赖于MAC地址。在网络上，通信的双方在同一局域网（LAN）内的情况很少，通常是经过多台计算机和网络设备中转才能连接到对方。而在进行中转时，会利用**下一站中转设备的MAC地址**来搜索下一个中转目标。这时会采用**ARP(Address Resolution Protocol)协议**。ARP 是一种用以解析地址的协议，根据通信方的IP地址就可以**反查对应的MAC地址**

<img src="http://ww1.sinaimg.cn/large/006rAlqhly1g7ualrfehzj30ro0ngqhh.jpg" alt="image.png" style="zoom:50%;" />

### 确保可靠性的TCP协议

按层次分，TCP位于传输层，提供可靠的**字节流服务**，所谓的字节流服务是指，为了方便传输，将大块数据分割成以报文段为单位的数据包进行管理。而**可靠的传输服务是**指，能够把数据准确可靠地传给对方。

为了准确无误地将数据送达目标处，TCP协议采用了**三次握手策略**。用TCP协议把数据包送出去后，TCP不会对传送后的情况置之不理，它一定会向对方确认是否成功送达。握手过程中使用了 **TCP 的标志(flag)** ---**SYN(synchronize)个ACK(acknowledgement)** 

- 发送端首先发送一个带SYN标志的数据包给对方
- 接收端收到后，回传一个带有 SYN/ACK 标志的数据包以示传达确认信息
- 最后，发送端再回传一个带ACK标志的数据包，代表“握手结束”

> ⚠️注意：若在握手过程中某个阶段莫名中断，TCP协议会再次以相同的顺序发送相同的数据包

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7uaxh2i51j30p40ec11i.jpg)

## 负责域名解析的DNS服务

DNS（Domain NAme System）服务和HTTP协议一样位于**应用层**的协议。它提供**域名到IP地址**之间的解析服务

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7ub1m6yz4j30qi0goak0.jpg)

## 各种协议与HTTP协议的关系的TCP/IP

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7ub3aonktj30qo11yh78.jpg)

## URI 和 URL

|      | URI                                                          | URL                                                          |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 定义 | 统一资源标识符                                               | 统一资源定位符                                               |
| 解释 | Uniform：规定统一的格式可方便处理各种不同类型的资源，而不用根据上文环境(类似文件路径)来识别资源指定的访问方式**(http:或ftp:、mailto、telnet、file)**<br />Resource:除了文档文件、图像或服务能够区别于其他类型的，全都可作为资源<br />Identifier：表示可标识的对象。也称为标识符 | 相对URL，是指从游览器中基本URI处指定URL，如/image/log.gif<br />绝对URL，使用协议方案名+登陆信息(可选)+服务器地址+服务器端口号+带层次的文件路径+查询字符串+片段标识符 |
| 实例 | ftp://ftp.is.co.za/rfc/rfc1808.txt<br /> http://www.ietf.org/rfc/rfc2396.txt<br />ldap://[2001:db8::7]/c=GB?objectClass?one<br /> mailto:John.Doe@example.com <br />news:comp.infosystems.www.servers.unix<br /> tel:+1-816-555-1212 <br />telnet://192.0.2.16:80/ urn:oasis:names:specification:docbook:dtd:xml:4.1.2<br/><br/> | http://user:pass@www.example.jp:80/dir/index.html?uid=1#ch1  |

