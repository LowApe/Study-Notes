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
