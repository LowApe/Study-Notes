作业
1：编译和解释的定义<br>
2：jdk、jre、jvm的定义和区别<br>
3：标识符的命名规则和遵循的原则<br>
4：基本数据类型有哪些？<br>
5：注释类型有哪些？<br>
6：jdk环境变量如何配置？<br>
附加7：查询一下jdk1.8、jdk1.9、jdk1.10、jdk1.11各自的新增功能

----


> 1.编译器是把源程序的每一条语句都编译成机器语言,并保存成二进制文件,这样运行时计算机可以直接以机器语言来运行此程序,速度很快;<br>
而解释器则是只在执行程序时,才一条一条的解释成机器语言给计算机来执行,所以运行速度是不如编译后的程序运行的快的.

----

> 2. JDK>JRE>JVM <br>
JVM ：英文名称（Java Virtual Machine），就是我们耳熟能详的 `Java 虚拟机`。它只认识 xxx.class 这种类型的文件，它能够将 class 文件中的字节码指令进行识别并调用操作系统向上的 API 完成动作。所以说，jvm 是 Java 能够跨平台的核心，具体的下文会详细说明。 <br>
JRE ：英文名称（Java Runtime Environment），我们叫它：`Java 运行时环境`。它主要包含两个部分，jvm 的标准实现和 Java 的一些基本类库。它相对于 jvm 来说，多出来的是一部分的 Java 类库。 <br>
JDK ：英文名称（Java Development Kit），`Java 开发工具包`。jdk 是整个 Java 开发的核心，它集成了 jre 和一些好用的小工具。例如：javac.exe，java.exe，jar.exe 等。

----

> 3. 标识符由大小写字母, 下划线, 数字, $符号组成.
2>开头可以是大小写字母, 下划线, 和$符号.(数字不能开头)
3>标识符长度没有限制
4>标识符不能是关键子和保留字

----

> java的基本数据类型有八种：
1. 四种整数类型(byte、short、int、long)：
2. 两种浮点数类型(float、double)：
3. 一种字符类型(char)：
4. 一种布尔类型(boolean)：true 真  和 false 假。

----

> 1. //
2. /** */
3. /**/

----

> 1. 路径
2. %JAVA_HOME%/bin or /jre
3. classPath：%JAVA_HOME%\lib\dt.jar %JAVA_HOME%\lib\tools.jar
----

> [JDK1.5，1.6，1.7，1.8，1.9，1.10，1.11的新特性整理](https://blog.csdn.net/J080624/article/details/85092655)
