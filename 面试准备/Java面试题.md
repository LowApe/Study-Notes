# Java 面试题

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
