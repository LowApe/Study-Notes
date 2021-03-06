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
- HashMap 可以把 null 作为key或者value，而HashTable 是不可以的
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
`java.util.concurrent` 包中提供了对线程优化、管理的各项操作，使线程的使用变得得心应手。该包提供了线程的运行，线程池的创建，线程生命周期的控制。

## Java 通过 `java.util.concurrent.Executors` 提供了四种静态方法创建四种线程池：
- newCachedThreadPool 创建一个**可缓冲的**线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程

- newFixedPool 创建一个长线程池，可**控制线程最大并发数**，超出的线程会在队列中等待

- new ScheduledThreadPool 创建一个定长线程池，支持**定时及周期性**任务执行

- newSingleThreadExecutor 创建一个**单线程化**的线程池，它只会用唯一的工作线程来执行任务，保证所以任务按照指定顺序(FIFO,LIFO,优先级)执行

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

## 简单说一下 servlet 的生命周期
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
3. forward 是一次请求中完成，而redirect 是重新请求

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
- cookie 是可以在客户端**禁用**的，这时候我们要使用 **cookie+数据库** 的方式存储，当从cookie 取不到，就从数据库

## MVC 的各个部分都有那些技术来实现？
M:javaBean<br>
V:html jsp <br>
C:controller servlet <br>

经典的MVC ：JSP+Servlet+JavaBean


# 数据库

| 关系型数据库     | 非关系型数据库     |
| :------------- | :------------- |
| MySQL<br>SQLServer<br>Oracle <br>...      |Redis<br>mogodb<br>hadoop<br>...      |

## 关系型数据库的三范式
范式就是规范，就是关系型数据库设计表时，要遵循的三个范式

| 范式     | 描述     |
| :------------- | :------------- |
| 第一范式       |   数据库表的每一列都是不可分割的基本数据项，同一列中不能有多个值，即实体中的某个属性不能有多个值或者不能有重复的属性(`列的不可分割性`)    |
| 第二范式       |   数据库表中每个实例或行必须可以被唯一地区分的唯一标识(`主键`)      |
| 第三范式       | 要求一个数据库表中不包含已在其他表中已包含的非主键字信息(`外建`)|


> 要想满足第二范式必须满足第一范式，要满足第三范式必须先满足第二范式

反三范式，有时候为了效率，可以设置重复的或者可以推导出的字段。如：订单(总价)和订单项(单价)，如果订单项很多，再去计算总价效率是不高的。

## 事务的四大特征ACID
事务是并发控制的单位，是用户定义的一个操作序列。这些操作要么都做，要么都不做，是一个不可分割的工作单位

| 特征 | 描述    |
| :------------- | :------------- |
|    原子性    |   表示事务内操作不可分割，要么都成功，要么都失败     |
|    一致性    |    事务操作后，数据不会破坏  |
|    持久性    |    事务中的所有数据操作必须被持久化到数据库中   |
|    隔离性    |    不同的隔离级别对应不同的干扰程度    |

## mysql 数据库的默认的最大连接数
一个数据库，特定服务器上的数据库只能支持一定数量**同时连接**，这时候需要设置最大连接数(最多同时服务多少连接），在数据库安装时都会有一个默认的最大连接数。

```mysql
show variables like 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+

# 设置最大连接数
set global max_connections=xxx
```

## mysql和oracle的分页语句
因为不可能完全显示数据，所以需要分页显示

| MySQL     | Oracle     |
| :------------- | :------------- |
| Mysql 是使用关键字 limit 来进行分页的 limit offset,size 表示从此索引开始，到多少大小       | Oracle 分页要使用三层嵌套查询，记不清楚  |
| String sql = "select * from students order by id limit "+ pageSize*(pageNumber-1)+","+pageSize； | String sql = "select * from"+(select *,rownum rid from(select * from students order by positive desc)where rid<="+pageSize*pagenumber+")as t" +"where t>"+pageSize* *(pageNumber-1); |



## 数据库的触发器的使用场景

需要有触法条件，当条件满足以后做什么操作。比如校园网、facebook你发一个日志，自动通知好友，其实就是在**增加日志时做一个触发**，用触发效率就很高。



```mysql
create table board1(id int primary key auto_increment,name varchar[50],articleCount int);

create table article1(id int primary key auto_increment,title varchar[50],bid int references board1(id));

delimiter | #把分割符，改成|
create tigger insertArticle_Trigger after insert on aricle1 for each row begin 
-> update board1 set articleCount=articleCount+1 where id=NEW.bid;
-> end;
-> |

delimiter;

insert into article1 value(null,'test',0);

insert into article1 value(null,'test',1);
```

> 设置了触发器，当插入语句，触发某字段+1

## 数据库的存储过程的使用场景

数据库存储过程具有的优点：

1. 存储过程只在创建时进行编译，以后每次执行存储过程都不需要重新编译，而一般SQL语句每执行一次就编译一次，因此使用存储过程可以大大提高数据库执行速度。
2. 复杂的业务逻辑需要更多条 SQL 语句。这些语句从客户机发送到服务器，当客户机和服务器之间的操作过多，将会产生大量网络传输，如果将这些操作放到存储过程中，网络传输就会大大减少，降低了网络负载。
3. 存储过程创建一次便可以重复使用，从而减少数据库开发人员的工作量
4. 安全性高，存储过程可以**屏蔽底层数据库对象**的直接访问，使用Execute 权限调用存储过程，无需拥有访问底层数据库对象的显式权限 

定义存储过程

```mysql
create procedure insert_Student(_name varchar(50),_age int,out_id int)
begin
	insert into student value(null,_name,_age);
	select max(stuId) into _id from student;
end;

call insert_Student('wfs',23,@id);
select @id;
```

## JDBC调用存储过程

- 加载驱动
- 创建数据库连接
- 创建Statement对象
- 执行SQL获取数据
- 释放连接

## 常用SQL语句

## JDBC的理解

Java Database Connection 数据库连接，数据库管理系统(mysql、Oracle)是很多，每个数据库管理系统的底层操作不一样的。

- 对于普通开发人员不知道 mysql 或 oracle 等数据库厂商的私有语言
- 就算知道，没用使用所有的数据库，需要定制很多代码

所以现在提供一个**接口(JDBC驱动)**，让数据库厂商自己实现接口，对于开发者而言，只需要导入对应厂商开发的实现，然后以接口方法进行调用。

## 写一个简单的JDBC的程序

```java
//STEP 1. Import required packages
import java.sql.*;

public class FirstExample {
    // JDBC driver name and database URL
    static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    static final String DB_URL = "jdbc:mysql://localhost/emp";

    //  Database credentials
    static final String USER = "root";
    static final String PASS = "123456";

    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        try{
            //STEP 2: Register JDBC driver
            Class.forName(JDBC_DRIVER);

            //STEP 3: Open a connection
            System.out.println("Connecting to database...");
            conn = DriverManager.getConnection(DB_URL,USER,PASS);

            //STEP 4: Execute a query
            System.out.println("Creating statement...");
            stmt = conn.createStatement();
            String sql;
            sql = "SELECT id, first, last, age FROM Employees";
            ResultSet rs = stmt.executeQuery(sql);

            //STEP 5: Extract data from result set
            while(rs.next()){
                //Retrieve by column name
                int id  = rs.getInt("id");
                int age = rs.getInt("age");
                String first = rs.getString("first");
                String last = rs.getString("last");

                //Display values
                System.out.print("ID: " + id);
                System.out.print(", Age: " + age);
                System.out.print(", First: " + first);
                System.out.println(", Last: " + last);
            }
            //STEP 6: Clean-up environment
            rs.close();
            stmt.close();
            conn.close();
        }catch(SQLException se){
            //Handle errors for JDBC
            se.printStackTrace();
        }catch(Exception e){
            //Handle errors for Class.forName
            e.printStackTrace();
        }finally{
            //finally block used to close resources
            try{
                if(stmt!=null)
                    stmt.close();
            }catch(SQLException se2){
            }// nothing we can do
            try{
                if(conn!=null)
                    conn.close();
            }catch(SQLException se){
                se.printStackTrace();
            }//end finally try
        }//end try
        System.out.println("There are so thing wrong!");
    }//end main
}//end FirstExample

```

- oracle 驱动`oracle.jdbc.driver.OracleDriver`
- 创建连接`DriverManager.getConnection(Driver,user,pass)`
- 设置参数`statement.setXXX(sql,value)`
- 执行`.executeXXX()`
- 释放连接 `.close()`

## JDBC中的PreparedStatement 相比 Statement的好处

- PreparedStatement 是预编译的，比Statement快
- 代码的可读性和可维护性
- 安全性(PreparedStatement 可以防止SQL注入攻击，而Statement却不能)

## 数据库连接池的作用

1. 限定数据库连接的个数，不会导致由于数据库连接过多导致数据库运行缓慢、崩溃
2. 数据库连接池不需要每次都去创建或销毁，节约了资源
3. 数据库连接池池不需要每次都去创建，响应时间更快

# 前端部分

## 简单说一下html、css、javascript 在网页开发定位

- html 定义网页结构
- css 层叠样式表 美化页面
- javascript 动态交互(ajax)

## 简单介绍Ajax

|      | AJAX [[/ˈeˌdʒæks/](cmd://Speak/_us_/AJAX)]                   |
| ---- | ------------------------------------------------------------ |
| 定义 | 异步的js和 xml                                               |
| 作用 | 通过AJAX与服务器进行数据交换，AJAX可以网页实现局部更新，这意味着不需要刷新页面，对页面某部分进行更新 |
| 实现 | 使用 XmlHttpRequest 对象，使用这个对象可以异步向服务器发送请求，获取响应，完成局部更新。 |
| 场景 | 登陆失败时不跳转页面<br />注册时提示用户是否存在<br />二级联动等待场景 |

## JS与JQuery的关系

- JQuery 是一个js框架，封装了**js的属性和方法**，并且增强了js的功能，让用户使用起来更加便利
- js是要处理很多兼容性的问题，由JQuery分装了底层,就不用处理兼容性问题
- 原生的js的dom、事件绑定和Ajax等操作非常繁琐，JQuery分装以后操作非常方便

```js
document.getElementById('idName') 

$('#idName')
```

## JQuery 的常用选择器？

- Id选择器 通过 Id 获取一个元素
- Class 选择器 通过的类获取元素
- 标签选择器 通过标签获取元素
- 通用选择器 * 获取所有元素
- 组合选择器
    - 儿子选择器 > 获取下面的子元素
    - 后代选择器 空格 获取下面后代，包括儿子、孙子等后代
- 属性选择器 Tag[attrName=“test”]

## JQuery 的页面加载完毕事件？

很多时候，获取元素，必须等到该元素加载完成后才能获取，我们可以把 js 代码放到该元素的后面，但是这样就会造成 js 在我们的 body 中存在不好管理，一般**获取元素做操作**等所有页面加载完毕后所有的元素

```js
// 第一种 ready(fn) 表示的是页面结构被加载完毕后执行传入函数 fn
$(document).ready(function(){
  
}); 

// 第二种  当页面加载完毕后执行里面的函数
$(function[]{
  
  });
```

window.onload 的区别

1. jQuery 中的页面加载完毕事件，表示的是页面结构被加载完毕
2. Window.onload 表示的是页面被记载完毕。必须等到页面中的图片、声音、图像等远程资源被加载完毕后才调用，而 jQuery 只需要页面结构被加载完毕。

## JQuery 的 AJax 和原生 JS 实现 Ajax 有什么关系 ?

>  JQuery 中的 Ajax 也是通过原生的 js 封装的。封装完成后让我们使用起来更加便利，不用考虑底层实现或兼容性等处理

如果采用原生 js 实现Ajax 是非常麻烦的，并且每次都是一样的。如果不使用 JQuery 我们也要封装对象的方法和属性

## 简单说一下 Html5?

增加了一些画图、视频、web 存储等高级功能，但是 html5 有一个不好的地方，html5 强调语义。例如 header footer 等语义标签

## 简单说一下 CSS3？

对原来 css 功能增强，增强了动画

- 阴影
- 渐变
- 转换、移动、缩放、旋转等
- 动画
- 可以使用媒体查询来实现响应式

Css3 最大缺点是根据不同的游览器处理兼容性。

##  Bootsrap 是什么？

> bootstrap 是一个移动设备优先的 **UI 框架**，不需要写 css 和 js 因为这些已经封装，我们使用bootstrap 就能实现比较漂亮的交互性的页面，提高编写效率

- 表单
- 布局-栅格系统

# 框架部分

## 什么是框架?

> 解决一个**开放性问题**，而设计的具有一定的约束性的支撑结构，在此结构上可以根据具体问题扩展、安插更多组成部分，从而**更迅速和方便地构建**完整的**解决**问题的**方案**

## 简单介绍 MVC 模式

Model View Controller,最简单的，最经典就是 jsp + Servlet + JavaBean

- 控制器接受游览器请求
- 控制器调用 JavaBean 处理业务
- 完成业务通过 jsp 显示

## MVC框架

为了解决传统 MVC 模式(jsp+Servlet+JavaBean)而出现的框架

传统 MVC 模式问题

- 所有的 Servlet 和 Servlet 映射都要配置在 web.xml，如果项目太大，web.xml 就太庞大，并且不能实现模块化
- Servlet 的主要功能就是接受参数、调用逻辑、跳转页面，比如其他文件上床、字符编码等功能也要写在 Servlet 中，不能让 Servlet 功能单一
- 接受参数比较麻烦。不能通过 Model 接受参数，只能单一接受完成后转换封装 Model
- 跳转方式比较单一(forword、redirect)

常见的 MVC 框架

- Structs
- Structs2
- Spring MVC

## Strcts2 的执行流程？

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g82dusg0l8j30z412yh5u.jpg)

1. 游览器发送请求，经过一些列的**过滤器(Filter)**,到达核心过滤器FilterDispatcher(StrcutsPrepareAndExecuteFilter) 

2. 询问 **ActionMapper** 来决定这个请求是否调用某个 Action，如果需要，就会把请求的处理交给 **ActionProxy**

3. ActionProxy 通过 **Configuration Manager** 询问框架的**配置文件(struts.xml)** ，找到需要调用的 Action 类

4. ActionProxy 创建一个 **ActionInvocation 实例**，Action 执行完毕，找到对应的**结果集**,在调用 Action 的过程前后，涉及到相关**拦截器**
5. 通过结果集的 Name 知道对应的结果集来对游览器进行**响应**

## Struts2 中的拦截器，你都用它干什么？

Struts2 的功能都是通过拦截器执行的。通过动态配置方式，可以在执行 **Action 的方法前后**，通过系统拦截器实现，执行相关逻辑，**加入相关逻辑**完成业务。使用场景：

1. 用户登陆判断，在执行 Action 的前面判断是否已经登陆，如果没有登录就跳转登录页面
2. 用户权限判断
3. 操作日志

## SpringMVC的执行流程？

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g82e6q9nq5j30xk0jo7nf.jpg)



1. 用户向服务器发送请求，请求被 **DispatcherServlet 捕获**
2. DispatcherServlet 对请求 URL 进行解析，得到请求资源标识符 URI ，调用 **HandlerMapping** 获得该 Handler 配置的所有请求的相关
3. DispatcherServlet 根据获得的**Handler** ，选择一个合适的 HandlerAdapter，提取 Request 中的模型数据，**填充 Handler 入参**，开始执行 Handler(Controller),Handler 执行完成后，向 DispatcherServlet **返回一个 ModelAndView 对象**
4. 接收到 ModelAndView ，选择一个合适的 **ViewResolver**
5. 通过 ViewResolver 与 ModelAndView 结合，**来渲染视图**，DispatcherServlet 将渲染结果返回给客户端

核心控制器捕获请求、查找 Handler、执行 Handler 、返回 ModelAndVIew ，选择ViewResolver、结合渲染、返回



## SpringMVC 和 Struts2 的不同

1. 核心控制器不同
    - Servlet
    - Filter
2. 控制器
    - 基于方法设计
    - 基于对象
3. 管理方式
4. 参数传递
    - 方法的参数
    - ValueStack
5. 学习难度
6. interceptre 的机制
    - AOP方式
    - 拦截器
7. AJAX 请求
    - 直接返回数据，自动帮我们转换为 JSON 数据
    - 通过插件

## Spring 中的两大核心

Spring 是什么？

​	Spring 是 J2EE 应用程序框架，是轻量级的 IoC和 AOP 的**容器框架**，主要针对 javaBean 的生命周期进行管理的轻量级容器

|      | IOC(Inversion of Control)或DI(Dependency Injection)          | AOP(Aspect Object Programm)                                  |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 定义 | IOC 控制反转，<br />核心原理：就是配置文件+反射(或工厂)+容器 | AOP面向切面编程<br />核心原理：使用动态代理的方式在执行前后或出现异常后执行相关逻辑 |
| 例子 | 原来：Service 需要调用DAO，Service 就创建 DAO，控制权在 Service<br />现在：Spring 发现你 Service 依赖于 dao(需要dao)，就给你注入(控制权在IOC容器)<br /> | 事务处理<br />权限判断<br />日志<br />...                    |

## 事务的传播特性



1. Propagation_required：如果存在一个事务，则支持当前事务；如果没有事务，则**开启一个事务**
2. propagation_support：如果存在一个事务，支持当前事务，如果没有事务，则**非事务的执行**
3. propagation_mandatory：如果已经存在一个事务，支持当前事务,如果没有事务，则**抛出异常**
4. propagation_requires_new：总是开启一个新的事务，如果一个事务已经存在，则将这个存在的**事务挂起**
5. propagation_not_supported：总是非事务地执行，并**挂起任何存在的事务**
6. propagation_never：总是非事务地执行,如果存在一个事务，则**抛出异常**
7. propagation_nested：如果一个活动的事务存在，则运行在一个嵌套的事务中，如果没有活动事务,则TransactionDefinition，按 required 执行

## 事务的隔离级别

- 脏读
- 幻读
- 不可重复读
- 序列化

## ORM

对象关系映射（Object Relational Mapping）模式是为了解决面向对象与关系型数据库存在的互不匹配的现象的技术。

传统使用jdbc 操作 sql，但是存在很多不足，所以出现 ORM 框架：Hibernate，ibatis，springframework 三个核心原则

1. 简单：
2. 传达性：数据库结构被任何人都能理解的语言文档化
3. 精确性：基于数据模型创建正确标准化了的结构。



## Mybatis 与 Hibernate 区别

相同点：

- 都是 ORM 框架、屏蔽 jdbc 的底层访问细节

不同点：

| Mybatis                                                      | Hibernate                                 |
| ------------------------------------------------------------ | ----------------------------------------- |
| 自己在 xml 配置文件中写 sql语句，提供将结果集自动封装为实体对象的集合 | 自动生成sql语句，并执行返回结果 java 结果 |
| 处理复杂 sql 语句                                            | 不能处理复杂 sql 语句，不直接控制 sql     |
| 简单，面向 sql ，不用考虑对象间复杂的映射关系                | 复杂                                      |

## Hibernate 映射对象的状态

- 临时状态/瞬时状态(transient):刚刚 **new 语句创建**，没有被持久化，有无 id **不处于 session 中**，该对象为临时对象
- 持久化状态/托管状态(persisent)：已经被持久化，加入到 session 的缓存中.session 是没有关闭该状态的对象为**持久化对象**
- 游离状态/托管状态(detached)：已经被持久化，但**不处于 session 中**，该状态的对象为游离对象
- 删除状态(removed):对象有关联的 ID，并且在 Session 管理下，但是已经被**计划删除**，如果没有事务就不能删除

临时状态->游离状态->持久状态->删除状态

## 介绍一下Hibernate 缓存

为了提高访问速度，把磁盘或数据库访问变成**内存访问**

缓存分类

- 一级缓存 session 缓存，内置不能被卸载
- 二级缓存 sessionFactory 的缓存（默认关闭）

> Hibernate 中的缓存分一级缓存和二级缓存，一级缓存就是 session 缓存，是内置的不能被卸载。二级缓存是 sessionFactory 级别的缓存，从应用启动到应用结束，是可选的，默认没有二级缓存，需要手动开启

保存数据库的数据，需要同步更新内存中的数据，所以增加开销，所以存放的数据要：

什么样的数据放在缓存：

- 经常被修改的数据。帖子最后回复时间
- 经常被查询的数据。电商的地点
- 不会被并发访问的数据  
- 常量数据

> ⚠️注意：hibernate 的二级缓存默认不支持分布式缓存，使用 redis 缓存



## WebService 使用场景

> WebService 是一个 SOA(面向服务的编程)的架构，它是不依赖于**语言**，不依赖于**平台**，可以实现不同的语言间的相互调用，通过 Internet 进行基于 Http 协议的网络应用间的交互

1. 异构系统的整合
2. 不同客户端的整合游览器、手机端
3. 天气预报

自我理解，感觉就是提供 API 服务



