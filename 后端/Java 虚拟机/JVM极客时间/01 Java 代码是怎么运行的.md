# Java 代码是怎么运行的

## Java 代码不同种运行方式
1. 开发工具运行
2. 双击 jar 文件运行
3. 命令行中运行
4. 网页中运行

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0vl5q5ms4j30cv06174f.jpg)


**C++ 运行方式** <br>

**机器码** ：把代码直接编译成 CPU 所能理解的代码格式,如下图
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0vlhbowl9j30j50dogmw.jpg)

1. Q: 为什么 Java 要在虚拟机中运行呢?
2. Q: Java 虚拟机又是怎样运行 Java 代码?
3. Q: 它的运行效率如何?


## 1. 为什么 Java 要在虚拟机中运行呢?

- Java 是门高级程序语言,语法复杂,抽象程度高。直接在硬件上运行这种复杂的程序并不现实。需要进行转换。
- 设计一个面向 Java 语言特性的虚拟机,并通过`编译器`将 Java 程序转换成该虚拟机所能识别的`指令程序`,也称 Java 字节码。之所以这么取名,是应为 Java `字节码`指令的操作码(opcode)被固定为一个字节。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0vli85qgmj30ed04tjrp.jpg)

- Java 虚拟机可以由`硬件`实现[1],但更为常见的是在各个现有平台上提供`软件`实现. 这样做的意义在于,一旦一个程序被转换成 Java 字节码,那么便可以在不同平台上的虚拟机实现运行。

- 虚拟机拥有`托管环境`(Managed Runtime).这个托管环境能够代替我们处理一些代码中冗长而且容易出错的部分。其中最为人知的`自动内存管理`和`垃圾回收`,这部分内容催生一波垃圾回收调优业务。

- 托管环境还提供数组越界、动态类型、安全权限等动态检查,免于书写这些无关业务逻辑的代码

## 2. Java 虚拟机具体是怎样运行 Java 字节码的?

1. 将字节码文件加载到 Java 虚拟机。
2. 加载后,Java 类会被存放到方法区(Method Area)。
3. 实际运行时,虚拟机会执行方法区的代码
4. Java 虚拟机会将栈细分为
    - 面向 Java 方法的 Java 方法栈
    - 面向本地方法(C++ 写的 native 方法)的本地方法栈
    - 存放各个线程执行位置的 PC 寄存器

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0vlzxh87bj30ab08kq3t.jpg)

在运行过程中,每当调用进入一个 Java 方法,Java 虚拟机会在当前的线程的 **Java方法栈** 中生成一个**栈帧**,用以存放局部变量以及字节码操作数。这个栈的针大小是提前计算好的,而且 Java 虚拟机不要求栈帧在内存中连续分布

当退出当前执行的方法时,不管是正常返回还是异常返回,Java 虚拟机均会**弹出**当前线程的当前栈帧,并将其舍弃

从硬件角度,Java 字节码无法直接执行。因此,Java 虚拟机需要将字节码翻译成机器码

在 HotSpot 里面,上述翻译过程有两种形式:
1. 解释执行,即逐条将**字节码**翻译成**机器码**并执行。
2. 即时编译(Just-In-Time compilation,JIT),即将一个方法中包含的所有字节码编译成机器码后再执行。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0vmd88mzxj30bx07j0tm.jpg)

前者优势在于**无需等待编译**,而后者的优势在于**实际运行速度更快**。 Hotspot 默认采用混合模式,综合了解释执行和即时编译两者的优点。它会先解释执行字节码,而后将其反复执行的热点代码,以**方法为单位**进行即时编译


## 3. Java 虚拟机的运行效率究竟如何?

HotSpot 采用多种技术包括**即时编译**来提升启动性能以及峰值性能

即时编译建立在程序符合二八定律假设上:20%的代码占据80%的计算资源

理论上讲,即时编译后的 Java 程序的执行效率,是可能超过 C++ 程序的。因为与静态编译相比,即时编译拥有程序的运行时信息,并且能够根据这个信息作出相应的优化

> 举个例子,我们知道虚方法(重写)是用来实现面向对象语言多态性.对于一个虚方法调用,尽管它有很多个目标方法,但在实际运行过程中它可能只调用其中一个。

这个信息便可以被即时编译所利用,来规避虚方法调用的开销

为了满足不同用户场景的需求,HotSopt 内置了多个即时编译器:C1、C2 和 Graal。Graal 是Java 10 正式引入的实际性即时编译器。之所以引入多个即时编译器,是为了在编译时间和生成代码的执行效率之间进行取舍.
- C1 又叫 Client 编译器,面向的是对启动性能有要求的客户端 GUI 程序,采用的优化手段相对简单,因此编译时间较短

- C2 又叫 Server 编译器,面向的是对峰值性能有要求的服务器端编程,采用的优化手段相对复杂,因此编译时间较长,但同时生成代码的执行效率较高

从Java 7开始，HotSpot 默认采用分层编译的方式:热点方法首先会被C1编译，而后热点方法中的热点会进一步被C2编译。

为了不干扰应用的正常运行，HotSpot 的即时编译是放在额外的`编译线程`中进行的。HotSpot会根据`CPU`的数量设置`编译线程`的数目，并且按`1:2`的比例配置给C1及C2编译器。

在计算资源充足的情况下，字节码的解释执行和即时编译可同时进行。编译完成后的机器码会在下次调用该方法时启用，以替换原本的解释执行。

----
asmtools.jar

```cmd
java -jar asmtools.jar jdls 文件 > xxx.asm
```
```cmd
java -jar asmtools.jar jasm 文件 xxx.asm
```
