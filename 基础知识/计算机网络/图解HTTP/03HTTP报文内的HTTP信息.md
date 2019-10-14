# HTTP报文内的HTTP信息

本章内容

- 了解一下请求和响应时怎样运作的

## HTTP报文

用于HTTP协议交互的信息被称为HTTP报文。

- 请求端(客户端)的HTTP报文叫做请求报文
- 响应端(服务端)的叫做响应报文

HTTP分为**报文首部**和**报文主体**两块

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7xwvjeer8j309606igmo.jpg)

## 请求报文及响应报文的结构

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7xwwznz7mj30ns0aijxh.jpg)

> 请求报文(上)和响应报文(下)

| 请求报文                                          | 响应报文                                             |
| ------------------------------------------------- | ---------------------------------------------------- |
| 请求行：包括用于请求的方式，请求 URI 和 HTTP 版本 | 状态行：包含表明响应结果的状态码，原因短语和HTTP版本 |

4种首部：通用首部、请求首部、响应首部和实体首部

## 编码提升传输速率

HTTP在传输数据时可以按照**数据原貌**直接传输，但也可以传输过程中通过**编码提升传输速率**。但是，编码的操作需要计算机来完成，因此会消耗更多的CPU等资源。

### 报文主体与实体主体的差异

- 报文(message) 
    - 是 HTTP 通信中的基本单位，由 8 位组字节流组成，通过 HTTP 通信传输。
- 实体(entity)
    - 作为请求或响应的有效**载荷数据(补充项)**被传输，其内容由**实体首部**和**实体主体**组成。

HTTP 报文的主体用于传输请求或响应的实体主体。通常，报文主体等于实体主体。只有当传输中进行编码操作时，实体主体的内容发生变化，才导致它和报文主体产生差异

### 压缩传输的内容编码

向待发送邮件内增加附件时，为了使邮件容量变小，我们会先用 ZIP 压缩文件之后再添加附件发送。 HTTP 协议中有一种被称为**内容编码**的功能也能进行类似的操作

> 内容编码：指明应用在实体内容上的编码格式，并保持实体信息原样压缩。内容编码后的实体由客户端接收并负责解码

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7xy5rxe18j30ou0dyte7.jpg)

常用的内容编码有以下几种

- gzip( GNU zip )
- compress( UNIX 系统的标准压缩)
- deflate( zlib )
- identity(不进行编码)

### 分割发送的分块传输编码

在 HTTP 通信过程中，请求的编码实体资源尚未全部传输完成之前，游览器无法显示请求页面。在传输大容量数据时，通过把**数据分割**成多块，能够让游览器**逐步显示页面**。

这种把实体主体分块的功能称为**分块传输编码（ Chunked Transfer Codong）**

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7xyazd5h7j30p20fqdns.jpg)

## 发送多种数据的多部分对象集合

发送邮件时，我们可以在邮件里写入文字添加多份附件。这是因为采用了 **MIME（Multipurpose Internet Mail Extensions ），多用途因特网邮件扩展机制**，它允许邮件处理文本、图片、视频等多个不同类型数据。

> 例如，图片等二进制数据以 ASCII 码字符串编码的方式指定，就是利用 MIME 来描述标记数据类型。而在 MIME 扩展中会使用一种称为**多部分对象集合（Multipart）**的方式，来容纳多份不同类型的数据

相应地，HTTP 协议中也采纳了多部分对象集合，发送的一份报文主体内可含有多类型实体。通常是在图片或文本文件等上传实用

| 对象                 | 描述                                                         | 实例                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| multipart/form-data  | 在 Web 表单文件上传时使用                                    | Content-Type:multipart/form-data;**boundary=AaB03x**<br />--AaB03x<br />Content-Disposition:from-data;name="field1"<br />Joe Blow<br />--AaB03x<br />Context-Disposition:from-data;name="pics";filename="file1.txt"<br />Content-Type:text/plain<br />...(file1.txt的数据)... |
| multipart/byteranges | 状态码 206 （partial content，部分内容）响应报文包含了多个范围的内容时使用 | HTTP/1.1 206 Partial Content<br />Date:Fri 13 Jul 2020 12:12:12 GMT<br />Last-Modified: Fri,31 Aug 2010 12:12:12 GMT<br />Content-Type:multpart/byteranges;boundary=THIS_STRING_SEPARATES<br />--THIS_STRING_SEPARARES<br />Content-Type:application/pdf<br />Content-Range:bytes 500-999/8000 |

在 HTTP 报文中使用多部分对象集合时，需要在**首部字段加上 Content-type**

使用 boundary 字符串来划分多部分对象集合指明的各类实体。在 boundary 字符串指定的各个实体的起始行之前插入“--”(--AaB03等)，而在多部分对象集合对应的字符串的最后插入“--”标记作为结束

## 获取部分内容的范围请求

以前，用户不能使用高速带宽，当时，下载一个尺寸较大的图片或文件就已经很吃力了。如果下载过程中遇到网络中断的情况，那就必须重头开始。为了解决上述问题，需要一种**可恢复的机制**。所谓恢复是指从之前**下载中断处恢复**下载

要实现该功能需要指定**下载的实体范围**。像这样，指定范围发送的请求叫做范围请求(Range Request).

> 对一份 10000字节大小的资源，如果使用范围请求，可以只请求 5001～10000 字节内的资源。执行范围请求时，会用到首部字段 Range 来制定资源的 byte 范围

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7xzen485yj30pw0jedtl.jpg)

| 范围                                     | 格式                        |
| ---------------------------------------- | --------------------------- |
| 5001～10000                              | Range:bytes=5001-10000      |
| 从5001字节之后全部                       | Range:bytes=5001-           |
| 一开始3000字节和 5000-7000字节的多重范围 | Range:bytes=-3000,5000-7000 |

针对范围请求，响应会返回状态码 206 Partial Content 的响应报文。另外，对于多重范围的范围请求，响应会在首部字段 Content-Type 标明 multipart/byteranges 后返回响应报文。



## 内容协商返回最合适的内容

同一个 Web 网站有可能存在多份相同内容的页面。比如英文版和中文版的 Web 页面，它们内容上虽相同，但使用的语言却不同

当**游览器默认语言**为中文或英文，访问相同 URI 的 Web 页面时，则会显示对应的英语班或中文版的 Web 页面。这样的机制称为**内容协商**

内容协商机制时指客户端和服务端就**响应的资源内容**进行交涉，然后提供给客户端最为合适的资源。内容协商会以**响应资源的语言、字符集、编码方式**等作为判断的基准。

- Accept
- Accept-Charset
- Accept-Encoding
- Accept-Language
- Content-Language

内容协商技术有 3 种类型

| 类型                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| 服务器驱动协商( Server-driver Negotiation) | 有服务端进行内容协商。以请求的首部字段为参考，在服务器端自动处理。 |
| 客户端驱动协商(Agent-driven Negotiation)   | 由客户端进行内容协商。用户从游览器显示的可选列表中手动选择。还可以利用 js 脚本在 Web 页面上自动进行上述选择。比如按 **OS的类型或游览器类型**自行切换成 PC 版页面或手机页面 |
| 透明协商( Transparent Negotiation)         | 是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进行内容协商的一种方法 |

