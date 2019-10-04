# Java 基础部分

## 简单讲一下 Java 的跨平台原理
- 不同操作系统及其CPU所使用的指令集不同
- 不同平台使用特定编译器让平台识别
- 编译器是与平台相关的，编译后的文件也是与平台相关的
- 我们说的语言跨平台是编译后的文件跨平台
- C语言是编译执行的，编译器与平台相关，编译生成的可执行文件与平台相关
- Java是解释执行的，生成中间码的编译器与平台无关(产生的字节码是跨平台的)，生成的的中间码也与平台无关，中间码再有解释器解释执行，解释器是与平台相关的
- 所以需要JVM,Sun 公司为不同平台提供对应的JVM。所以，实现跨平台的根本机制还是JVM

> Java通过不同的系统、不同版本、不同位数的 Java虚拟机，来屏蔽不同的系统指令集差异而对外统一的接口。

## Java 基本数据类型
8种：
- byte 1字节 8位
- short 2字节 16位
- int 4字节 32位
- long 8字节 64位
- char 2字节 16位
- float 4字节 32位
- double 8字节 64位
- boolean 1字节 8位

## 面向对象的特征
四大基本特征：封装、抽象、继承、多态

- 封装
  - 将对象封装成一个**高度自治和相对封闭**的个体，对象自己操作自己的行为与状态
- 抽象
  - 将现实生活中的对象，抽象成类
- 继承
  - 把已经存在的类所定义的内容作为自己的一部分，并可以加入若干的新内容，也可以修改父类使之更合适
- **多态**
  - 同一行为具有不同的表现形式。即同一行为（消息）可以根据发送对象的不同采取多种不同的行为

> 动态绑定：指在运行期间判断所**引用对象的实际类型**，根据其实际类型调用相应的**方法** <br/>
多态三个必要条件：继承、重写、**父类引用指向子类对象**

## 装箱和拆箱

存在于基本数据类型与包装类型
- 装箱：将基本数据类型转为对应的包装类型
    - 自动装箱：在编译时会调用Integer.valueOf(1)
- 拆箱：将包装类型转化为对应的基本数据类型
    - 自动拆箱：在编译时会调用i.intvalue()

```java
//自动装箱和拆箱
Integer i = 5;
int j = i;
//手动装箱和拆箱
Integer i = Integer.valueOf(5);
int j = i.intValue();
```

Q：有了基本数据类型，为什么还需要包装类型？<br>
A：Java是一个面向对象的语言，而基本的数据类型，不具备面向对象的特性

## == 与 equals 的区别

- ==：判断两个变量之间的**值**是否相等，变量分为
    - 基本数据类型变量，直接比较**值**
    - 引用类型，比较对应的引用内存的**首地址**
- equals：用来比较两个对象的某个特性是否一样，实际上就是调用对象的 equals 方法。不同对象 equals 方法可以复写

```java
//Object equals()方法
public boolean equals(Object obj) {
    return (this == obj);
}
```

## String

Q：String 和 StringBuilder 的区别(final)?<br>
A：
- String 是内容不可变的字符串。String底层是不可变的**字符数组**(final 修饰的 char[])
- StringBuilder StringBuffer(线程安全)是内容可变的字符串。底层是可变的字符数组

> Java 中提供三个类String StringBuilder StringBuffer 来表示和操作字符串，字符串就是多个字符的集合

> final 修饰的类不能被继承，修饰的变量不能被修改

**最经典的拼接字符串**
- String 进行拼接 String c="a"+"b"(初始化c，c只能赋值一次)
- StringBuilder 或者 StringBuffer    
    - StringBuilder sb = new StringBuilder();
    - sb.append("a").append("b")

> String 拼接需要创建好几个对象，StringBuilder 或 StringBuffer 很容易拼接字符串，效率高，可以根据大小增加容量

> StringBuffer是线程安全的，但是效率较低，StringBuilder是线程不安全的，效率较高

## Java的集合
Java 集合
- 存值
    - List
    - Set
- 存 key-value
    - Map

### List 和 Set 的区别？
- List 是有序的，可以重复的
- Set 是无序的，不可以重复的。
    - 根据equals 和 hashcode 判断
    - 如果一个对象存储在 Set 中，必须重写 equals 和 hashcode 的方法

### ArrayList 和 LinkedList 的区别？
- ArrayList 底层使用数组
- ListedList 底层使用链表

> 也就是比较两种数据结构的差距<br>
ArrayList 使用查询比较多，但是插入和删除比较少的情况。<br>LinkedList 使用查询比较少，而插入和删除比较多的情况

|  | 数组     | 链表|
| :------------- | :------------- |:------------- |
| 读取       | O(1)       | O(n)|
| 插入       | O(n)       | O(1)|
| 删除       | O(n)       | O(1)|

### HashMap 和 HashTable 的区别？
- 两个都可以存储 key-value
- HashMap 可以吧 null 作为key或者value，而HashTable 是不可以的
- HashMap 是线程不安全的，效率较高，而HashTable 是线程安全的，效率较高

### HashTable 和 ConcurentHashMap(线程安全又效率高) 的区别？

ConcurrentHashMap：通过把整个 Map 分为 N 个 Segnent(段，类似于HashTable)，可以提供相同的线程安全，但是效率提升 N 倍，默认提升 16 倍

## 实现一个拷贝文件的工具类使用字节流还是字符流？
我们拷贝的文件不确定是只包含字符流，有可能有字节流(图片、声音、图像等),为了考虑到通用性，要使用字节流

## 线程的几种实现方式？
- 通过继承 Thread 类实现一个继承
- 通过实现 Runnable 接口实现一个线程

> 继承扩展性不强，Java总只支持单继承，如果一个类继承 Thread 类，就不能继承其他的类了。

启动：<br>
```java
Thread thread = new Thread(继承了 Thread 的对象/实现了 Runnable 的对象);
thread.start();
// 启动线程使用 start 方法，而启动了以后执行的是 run 方法
```
区分线程：<br>
在一个系统中有很多线程，每个线程都会打印日志，我们想区分是哪个线程.thread.setName("线程名");
> ⚠️注意：这是一种规范，在创建线程完成后，都需要设置名称

## 有没有使用过线程并发库？
简单了解过<br>
JDK5 中增加了 DougLea 的并发库，这一引进给Java线程的管理和使用提供了强大的便利性。<br>
`java.util.current` 包中提供了对线程优化、管理的各项操作，使线程的使用变得得心应手。该包提供了线程的运行，线程池的创建，线程生命周期的控制。

## Java 通过 `java.util.concurrent.Executors` 提供了四种静态方法创建四种线程池：
- newCachedThreadPool 创建一个**可缓冲的**线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程

- newFixedPool 创建一个长线程池，可控制线程最大并发数，超出的线程会在队列中等待

- new ScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行

- newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所以任务按照指定顺序(FIFO,LIFO,优先级)执行

## 线程池的作用
1. 限定线程的个数，不会导致由于线程过多导致系统运行缓慢、崩溃
2. 线程池不需要每次都去创建或销毁，节约了资源
3. 线程池不需要每次都去创建，响应时间更快

## 什么是设计模式？常用的设计模式有哪些？
设计模式就是经过**无数前人的实践总结出得**，设计过程中可以**反复使用的**、可以解决特定问题的**设计方法**

- 单例模式(饿汉式、懒汉式)
    1. 构造方法私有化
    2. 创建私有静态对象实例
    3. 提供公有静态方法获取开放接口
- 工厂模式:Spring IOC 就是使用了工厂模式
- 代理模式:Spring AOP 使用动态代理
- 装饰者模式:Java IO

# Java Web

## http 中 get 和 post 请求的区别?
- Get 和 Post 请求都是 http 的请求方式，用户通过不同的 http 的请求方式完成对资源(url)操作。
- GET、POST、PUT、DELETE 对应着对这个资源的查、改、增、删4个操作。
- Get用于获取/查询资源，而Post用于更新资源
- Get 请求提交的资源会在**地址栏显示**出来，而 post请求不会在地址栏显示，二是在Http包的包体
- Get 请求由于游览器对**地址长度的限制**而导致传输的数据有限制，而Post请求没有长度限制
- 安全性，Post的**安全性**要比Get的安全性高

## 说一下你对 servlet 的理解？servlet是什么？

Servlet 是用Java编写的服务器端程序，而这些 Servlet 都要实现 Servlet 这个接口，主要功能在于交互式地**游览和修改**数据，生成动态 Web 内容。HttpServlet 重写 doGet 和 doPost 方法或者重写 service 方法完成对 get 和 post 请求的响应

## 简单好一下 servlet 的生命周期
- 加载和实例化 （构造方法 init）
- 处理请求以及服务结束 （service、destory方法）

> 请求到达 service 方法，service 方法自动派遣运行与请求对应的 doXXX方法(doGet、doPost)，当服务器决定将实例化销毁的时候(服务器关闭)调用其 destory 方法

加载Servlet的class<br>
->实例化Servlet<br>
->调用Servlet的init()完成初始化<br>
->响应请求(service方法)->Servlet容器关闭调用 destory 方法

## Servlet 中 forward() 与 redirect()的区别?
- forward 是**容器(服务端)中控制**的转向，在客户端游览器地址中不会显示出转向的地址，并重新发送**原来的请求**，在服务端的效率比较高效，因为不用重新发请求
- redirect 是完全的跳转，游览器会显示跳转的地址，并**重新发送新的请求**
- 有些情况下，需要跳转到其他服务器上的资源，则必须使用 **sendRedirect()方法**

1. forward 是服务器端的转向而 redirect 是客户端的跳转
2. forward 游览器地址不会发生改变，而 redirect 会发生变化
3. forward 是一次请求中完成，而redirect 是重新

## JSP 内置对象？作用分别是什么？分别有什么方法？
9大内置对象：
- request 用户端请求
- response 网页传回用户端的回应
- pageContext 网页属性管理
- session 与请求有关的会话
- application servlet正在执行的内容
- out 用来传送回应的输出
- config
- page JSP页面本身
- exception 针对错误网页，未捕捉的例外

四大作用域:
- request
- pageContext
- session
- application

> 通过 jstl 从四大作用域中取值

## session 和 cookie 的区别？你在项目中都有那些地方使用了？
- session 和 cookie 都是会话跟踪技术。
- Cookie 通过在**客户端**记录信息确定用户身份
- Session 通过在**服务器端**记录信息确定用户身份
- 但是 Seesion 的实现依赖于 Cookie，sessionId(session 的唯一标识存放在客户端)

1. cookie 数据存放在客户的游览器，session 数据放在服务器上
2. cookie 不是很安全，容易被Cookie 欺骗，考虑到安全应当使用 session
3. session 会在一定时间内保存在服务器上，当访问过多，会比较**占用服务器资源**，考虑服务器性能，使用 Cookie
4. 单个 cookie 保存的数据不能超过 4K，很多游览器都限制一个站点最多保存 20 个cookie
5.
- 将登陆信息等重要信息保存为 Session
- 其他信息如果需要保留，可以放在Cookie(购物车)
- cookie 是可以在客户端**禁用**的，这时候我们要使用 **cookie+数据库** 的方式实现，当从cookie 取不到，就从数据库

## MVC 的各个部分都有那些技术来实现？
M:javaBean<br>
V:html jsp <br>
C:controller servlet <br>

经典的MVC ：JSP+Servlet+JavaBean
