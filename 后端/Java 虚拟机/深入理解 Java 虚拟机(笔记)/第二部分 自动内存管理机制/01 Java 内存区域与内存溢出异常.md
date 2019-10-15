# Java 内存区域与内存溢出异常

## 概述
Java 与 C++ 之前有一堵由**内存动态分配**和**垃圾收集技术**所围成的高墙


|| C、C++ 程序员     | Java程序员     |
|:------| :------------- | :------------- |
|特点| 拥有每一个对象的"所有权"       | 虚拟机**自动内存管理机制**       |
|优势|可以精准控制|不需要为每个对象编写 delete/free 代码，不容易出现**内存泄漏**和**内存溢出**|
|劣势|操作不当，容易内存泄漏和内存溢出|一旦出现内存泄漏和内存溢出，不了解虚拟机是怎么使用内存的，那么**排查**异常会非常**困难**|


## 运行时数据区域

Java虚拟机在**执行**Java程序的过程中会把它管理的**内存划分**为若干个不同的**数据区域**,这些区域都有各自的用途，以及创建和销毁的时间，有的区域随着**虚拟机进程的启动**而存在，有些区域则依赖**用户线程的启动和结束**而创建和销毁。

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7pgehor73j30m80igq6n.jpg)

### 程序计数器
程序计数器是一块较小的内存空间，它可以看作是**当前线程**执行的**字节码的行号指示器**

在虚拟机的概念模型里，字节码解释器工作时就是通过**改变计数器的值**来选择下一条需要执行的**字节码指令**。分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器

> 由于Java虚拟机的多线程是通过**线程轮流切换**并**分配**处理器**执行时间**的方式来实现的，在任何一个确定的时刻，一个处理器(对于多核处理器来说是一个内核)都**只会执行一条线程中的指令**。因此线程切换后恢复到正确的执行位置，都需要有独立的程序计数器，**各线程之间计数器互不影响**，独立存储，我们称这类内存区域为**线程私有**的内存

> ⚠️提示：如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的**虚拟机字节码指令的地址**；如果正在执行的是 Natvie 方法，这个计数器则为空（Undefined）。此内存区域是**唯一一个**在Java虚拟机规范中没有规定任何 `OutOfMemoryError`情况的区域

### Java虚拟机栈
与程序计数器一样，Java虚拟机栈也是**线程私有**的，它的生命周期与线程相同。

虚拟机栈描述的是Java**方法执行**的内存模型：每个方法执行的同时都会创建一个**栈帧(Stack Frame)** 用于存储 `局部变量表`、 `操作数栈` 、`动态链接`、`方法出口`等信息。每个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中**入栈到出栈**的过程

局部变量表存放了**编译期**可知的各类`基本数据类型`、`对象引用类型`、`returnAddress类型`

> ⚠️提示：其中64位长度的 long和double类型的数据会占用2个局部变量空间，其余的数据类型只占用1个。局部变量表所需要的内存空间`在编译期间完成分配`，当进入一个方法时，这个方法需要的`帧`中分配多大的`局部变量空间是完全确定的`，在方法运行期间不会改变局部变量表的大小

Java区域规定了两种异常情况：
- 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 `StackOverflowError`异常
- 如果虚拟机可以动态扩展时无法申请到足够的内存空间，则会抛出 `OutOfMemoryError`异常

### 本地方法栈
本地方法栈与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行Java方法方法(**也就是字节码**)服务，而本地方法栈则为虚拟机使用的**Native方法**服务

> ⚠️注意：在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机(譬如Sun 的HotSot虚拟机)直接把本地方法栈与虚拟机栈合二为一

### Java 堆
Java堆(Java Heap)是Java 虚拟机所管理的内存中最大的一块。Java堆是被所有**线程共享**的一块内存区域，在**虚拟机启动时创建**。此内存区域的唯一目的就是**存放对象实例**，几乎所有的对象实例都在这里**分配内存**

> ⚠️注意：在Java虚拟机规范中这样描述：所有的对象实例以及数组都在`堆上分配`，但是随着JIT编译器的发展与`逃逸分析`技术逐渐成熟，`栈上分配`、`标量替换`优化技术将会导致不一定所有的对象都在堆上分配

Java 堆是垃圾收集管理的主要区域，因为很多时候也被称为`GC堆`，用于现在收集器基本都采用**分代收集算法**，所有Java堆还可以细分：新生代与老生代：在细致点的有 Eden空间、From Survivor 空间、To Surviror 空间、ToSurvior 空间等。从分配的角度来看，线程共享的Java堆中可能划分出**多个线程私有的**分配缓冲区TLAB。

无论如何划分，都与存放内容无关，无论哪个区域，存储的都仍然是对象实例。进一步划分的目的是为了更好的**回收内存**，或更快地**分配内存**

> ⚠️注意：Java堆可以处于物理上**不连续的内存**空间，只要**逻辑连续**。当前主流的虚拟机都是按照可扩展来实现的(通过-Xmx和-Xms来控制)。如果在堆中没有内存完成实例分配，并且堆也无法扩展时，将会抛出`OutOfMemoryError`异常

### 方法区

方法区与Java堆一样，是**各个线程共享**的内存区域，他用于存储已被**虚拟机加载**的`类信息`、`常量`、`静态变量`、`即时编译器编译后的代码`等数据。虽然Java虚拟机规范把方法区描述为**堆**的逻辑部分，其目的是将Java堆区分开来。

习惯于 HotSpot 虚拟机上开发、部署程序的开发者来说，很多人更愿意把方法区称为"永久代"，本质上两者并不等价，仅仅是因为 HotSpot 虚拟机的设计团队选择把 **GC分代收集扩展** 至方法区，或者说使用永久代来实现方法区而已，这样HotSpot的垃圾收集器可以像管理Java堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作

> ⚠️注意：对于其他虚拟机(BEA JRockit、IBM J9)来说是不存在永久代来实现方法区，原则上，如果实现方法区属于虚拟机实现细节，不受虚拟机规范约束，但使用永久代来实现方法区，现在看来并不是一个好主意，因为这样更容易遇到内存溢出问题(永久代有 -XX：MaxPermSize的上限，J9和Rockit只要没有触碰到进程可用内存的上限，例如32位系统中的4GB，就不会出现问题)，而且有极少数方法(例如String.intern())会因这个原因导致不同虚拟机下有不同的表现。对于 HotSpot 虚拟机，根据官方发布的路线图信息，现在也有放弃永久代并逐步改为采用 `Native Memory`来实现方法区的规划，JDK1.7的 HotSpot 中，已经把原本放在永久代的`字符串常量池`移出


Java 虚拟机规范对方法区的限制非常宽松，除了和Java堆一样不需要连续的内存和可以选择固定大小或者扩展外，还可以选择**不实现垃圾收集**。相对而言垃圾收集在这个区域的内存是比较少出现的，但并非数据进入方法区就如用生代名字一样永久存在。这个区域的内存回收目标只要是针对**常量池的回收和对类型的卸载**

> Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出`OutOfMemoryError`异常

### 运行时常量池
运行时常量池(Runtime Constant Pool)是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是**常量池**，用于存放**编译期**生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的**运行时常量池**中存放

Java虚拟机对Class文件每部分格式都有严格规定。每个字节用于存储那种数据必须符合规范的要求才会被虚拟机认可、装载和执行。但对于运行时常量池，Java 虚拟机规范没有任何细节的要求，不同的提供商实现的虚拟机可以按照自己的需要实现这个内存区域。

运行时常量池相对于**Class文件常量池**的另一个重要特性是具备动态性，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能放入池中。例如：String.intern() 方法

> 运行时常量池属于方法区一部分，所以放常量池无法再申请到内存时，将抛出`OutOfMemoryError`异常


### 直接内存

直接内存既不是虚拟机运行时数据区域，也不是Java虚拟机规范定义的内存区域。但是这部分也是频繁使用，而且可能导致内存溢出异常

在JDK 1.4 新加入 NIO(New Input/Output)，引入一种**基于通道**与**缓存区**的I/O方式，它可以使用Native函数库直接分配**堆外内存**，然后通过一个存储在Java堆中的**DirectByteBuffer对象**作为这块内存的引用进行操作。这样能在一些场景中显著提供性能，因为避免在Java堆和Native堆中来回复制数据

> ⚠️注意：本机直接内存的分配并不会受到 Java 堆大小的限制，但是既然是内存，肯定还会受到本机总内存(包括RAM和SWAP区或者分页文件)大小及处理器寻址空间的大小。服务器管理员在配置`虚拟机参数时`，会根据实际内存设置 `-Xmx`等参数信息。如果忽略了直接内存，使得各个内存区域总和大于物理内存限制(包括物理的和操作系统级的限制)，从而导致`动态扩展`而造成`OutOfMemoryError`异常

## HotSpot 虚拟机对象揭秘

对象如何创建、如何布局以及如何访问的。涉及的细节将范围限定在**Java堆**中，

### 对象的创建

在虚拟机中，对象创建（不包括数组和Class对象）是一种什么过程？

1. 检查new指令的参数是否能在**常量池**中定位到这个类的**符号引用**，并且检查这个类是否被加载、解析和初始化。如果没有则必须执行相应的类加载过程
2. 类加载完成后便可完全确定，为对象分类空间内存的大小。分配内存根据Java堆规整程度分为两种
    - 指针碰撞：如果规整，分配空间仅仅把指针指向空闲空间那边挪动一段与对象大小相等的距离
    - 空闲列表：如果不是规整，从列表中找到一块足够大的空间划分给实例
3. 虚拟机对对象进行设置

> ⚠️注意：划分空间之外需要考虑，并发情况下并不是线程安全的。需要两种解决方案
>
> - 对分配空间的动作进行**同步处理**（CAS+失败重试）
> - 把内存分配的动作**按照线程划分在不同的空间**（TLAB Thread Local Allocation Buffer,本地线程分配缓冲）

4. 内存分配完成后，虚拟机需要将分配到内存空间都初始化为**零值**（不包括对象头）	
    - 如果使用TLAB，这一工作可以在TLAB分配时进行
5. 虚拟机对对象进行必要的设置
    - 例如对象是那个类的实例、如何找到类的元数据信息、对象的哈希码、对象的GC分代年龄等，这些信息放到对象的对象头中
6. 执行对象的`<init>`方法

### 对象的内存布局

对象的内存布局可以分为3个部分：

- 对象头
- 实例数据
- 对齐填充

|          | 定义                                             | 描述                                                         |
| -------- | ------------------------------------------------ | ------------------------------------------------------------ |
| 对象头   | 1. 用于存储对象自身的运行时数据<br />2. 类型指针 | 1. 如哈希码、GC分代年龄、锁状态标志、线程池有锁、偏向线程ID、偏向时间戳<br />2. 指向对象的元数据的指针(如果是Java数组，那么对象头还必须有一块记录数组的长度) |
| 实例数据 | 对象真正的存储有效信息                           | 存储顺序会受到Java源码中定义顺序或HotSpot 分配策略参数；从策略中可以看出，相同宽度的字段分配到一起 |
| 对齐填充 | 仅仅起着占位符的作用                             | HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍 |

### 对象的访问定位

Java程序需要通过**栈**上的reference数据表操作**堆**上的具体对象

> ⚠️注意：Java虚拟机规范中reference类型规定一个指向对象的引用，并没有定义这个引用通过何种方式去**定位、访问堆中的对象的具体位置**，所以访问方式取决于虚拟机实现而定

目前主流的访问方式有使用**句柄**和**直接指针**两种

- 如果使用句柄的话，那么Java堆将会划分出一块内存来作为**句柄池**，reference 存储的是对象的**句柄地址**，而句柄包含对象实例数据和类型数据各自的具体地址信息

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7v8hrmtlhj30k908idgk.jpg)

- 如果使用直接指针访问，那么Java堆对象的布局中就考虑如何设置访问类型数据的相关信息，而reference中存储的直接就是对象地址

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7v8nam5fwj30kk089wf5.jpg)

| 访问方式     | 优势                                                         |
| ------------ | ------------------------------------------------------------ |
| 句柄访问     | 对象被移动(垃圾收集时移动对象是非常普遍的行为)时只会改变句柄中的实例数据指针，而reference本身不需要修改 |
| 直接指针访问 | 速度更快，对象访问在Java中非常频繁，因此开销积少成多后也是非常可观的执行成本 |

## 实战：OutOfMemoryError 异常

本节内容的目的：

- 通过代码验证 Java 虚拟机规范中描述的各个运行时区域存储的内容
- 希望在工作中遇到实际的内存溢出异常时，能根据异常的信息快速判断是哪个区域的内存溢出，以及出现后如何处理

### Java堆溢出

Java堆用于存储对象实例，只要不断地创建实例，并且保证 GC Roots 到对象之间存在可达路径来避免垃圾回收机制清除对象，那么对象达到最大**堆的容量限制后**产生内存溢出异常

通过 Idea 写一个无限创建实例对象的类

```java
/**
 * VM Args:-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
 */
public class HeapOOM {
  //静态类
    static class OOMObject{

    }

    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while (true){
            list.add(new OOMObject());
        }
    }
}
```

VM 参数限制 Java 堆的大小为 20MB ，不可扩展(-Xms -Xmx 参数一样可避免**堆自动扩展**)，通过参数 `-XX:+HeapDumpOnOutOfMemoryError`可以让虚拟机在出现内存溢出异常时**Dump 出当前的内存堆转存快照**以便事后分析



![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1g7ypurdnrhj31ni11kav9.jpg)

**结果：**

```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid9140.hprof ...
Heap dump file created [27781436 bytes in 0.120 secs]
  
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:265)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:239)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:231)
	at java.util.ArrayList.add(ArrayList.java:462)
	at HeapOOM.main(HeapOOM.java:12)
```

**解决这个区域的异常：**

一般通过**内存映射分析工具**对 Dump 出来的堆转存快照进行分析，重点确认内存中的对象是否是必要的，也就是要先清楚到底是**内存泄漏(Memory Leak)**还是**内存溢出(Memory Overflow)**

<img src="http://ww1.sinaimg.cn/large/006rAlqhly1g7yw3nr672j31uo19oqv5.jpg" alt="image.png" style="zoom:50%;" />

> 如果是内存泄漏，可进一步通过工具查看泄漏对象到 GCRoots 的引用链。于是就能找到泄漏对象是通过怎样的路径与 GC Roots 相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息及 GC Roots 引用链的信息，就可以比较准确定位出泄漏代码的位置。

目前还不清楚 GC Roots 是什么...

### 虚拟机栈和本地方法栈溢出

由于在 HotSpot 虚拟机中并不区分虚拟机栈和本地方法栈，因此，对于 HotSpot 来说，虽然`-Xoss`参数(设置本地方法栈大小)存在，但实际上无效的，栈容量只由`-Xss`参数设定。关于虚拟机栈和本地方法栈，在 Java 虚拟机规范中描述了两种异常：

- 如果线程**请求的栈深度**大于虚拟机所允许的**最大深度**，将抛出**StackOverflowError**异常
- 如果虚拟机在**扩展栈时**无法申请到足够的内存空间，则抛出**OutOfMemoryError**异常

其中存在一些互相重叠的问题：当栈空间无法继续分配时，到底是内存太小，还是已使用的栈空间太大，其本质上只是对同一件事情的两种描述而已



```java
/**
 * VM args: -Xss128k 或者 -Xss160k
 */
public class JavaVMStackSOF {
    private int stackLength = 1;
    public void stackLeak(){
        stackLength++;
        stackLeak();
    }
    public static void main(String[] args) {
        JavaVMStackSOF javaVMStackSOF = new JavaVMStackSOF();
        try{
            javaVMStackSOF.stackLeak();
        }catch (Throwable e){
            System.out.println("stack length:" + javaVMStackSOF.stackLength);
            throw e;
        }
    }
}
```

**结果：**

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7yxpewxfnj30pk06w7c5.jpg)

> 其中发现 mac 10.13.6 下他要求我栈设置最少为 160k，然后结果居然 771，设置 -Xss161k 结果 7660！！！，我用windows -Xss128k 结果 998，设置 -Xss160k 结果 1734.设置栈的最大容量，并且使用**递归**，利用了栈的特性，我的理解，每执行一次方法如果生成一个栈帧，那由于递归将会使这个栈无限增大，然后造成扩展栈无法申请到足够空间然后抛出了 **OutOfMemoryError**，但是上述结果却**抛出StackOverflowError**

实验结果表明：**在单个线程**，无论是由于**栈帧太大还是虚拟机容量太小** ，当内存无法分配的时候，虚拟机抛出的都是**StackOverflowError**异常

> ⚠️注意：书上通过不断建立线程的方式倒是可以产生内存溢出异常，但是代码简直不能允许。。。windows 下直接死机，mac 下系统卡顿，没敢在往下运行.姑且记住多线程下导致的内存溢出

再不能减少线程数或者更换 64 位虚拟机的情况下，就只能通过**减少最大堆和减少栈容量**来获取更多的线程。如果没有这方面的处理经验，这种通过“减少内存”的手段来解决内存溢出的方法会比较难以想到

### 方法区和运行时常量池溢出

由于**运行时常量池**是方法区的一部分，因此这两个区域的溢出测试就放在一起进行。

String.intern() 是一个 Native 方法，它的作用是：如果字符串常量池中已经包含了一个等于此 String 对象的字符串，则返回代表池中这个字符串的 String 对象；否则，将此 String 对象包含的字符串添加到常量池中，并且返回此 String 对象的引用。在 JDK 1.6 及之前的版本中，由于常量池分配在永久代内，我们可以通过 `-XX:PermSize` 和 `-XX：MaxPermSize`限制方法区大小，从而间接限制其中常量池的容量。

```java
/**
 * VM Args： -XX:PermSize=10M -XX:MaxPermSize=10M
 */
public class RuntimeConstantPoolOOm {
    public static void main(String[] args) {
        //使用 List 保持着常量池引用，避免 Full GC 回收常量池行为
        List<String> list = new ArrayList<>();
        int i = 0;
        while(true){
            list.add(String.valueOf(i++).intern());
        }
    }
}
```

**结果：**

```java
Exception in thread "main" java.lang.OutOfMemoryError:PermGen space
```

JDK1.7 运行这段程序就不会得到相同的结果，while 循环将一直进行下去。

```java
public class RuntimeConstantPoolOOm {
    public static void main(String[] args) {
        String str1 = new StringBuilder("计算机").append("软件").toString();
        System.out.println(str1.intern() == str1);
        String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2.intern()==str2);
    }
}
```

这段代码在 JDK 1.6 中运行，会得到两个false，而在 JDK 1.7 中运行，会得到一个 true 和 false。产生的原因是：

- 在 JDK 1.6 中，intern() 方法会把首次遇到的字符串**复制**到永久代中，返回的也是永久代中这个字符串实例的引用，而由 StringBuilder 创建的字符串实例在 Java 堆上，所以必然不是同一个引用，将返回 false
- 而 JDK 1.7 (以及部分其他虚拟机，例如 JRockit)的 intern()实现**不会再复制**实例，只是在常量池中**首次出现的实例引用**，因此 intern() 返回的引用和 StringBuilder 创建的那个字符串实例是同一个。

> ⚠️注意：我的版本是 JDK 1.8，然后调整了中文和英文的顺序，然后结果是先false 然后 true，然后 我把英文里添加了中文，然后他就为 true true 了，好奇怪，后来尝试改变数值，发现 java 这个字符串在常量池应该已经存在了，所以我们第一次引用 instern() 比较false ，然后我切换了字符串不显中文还是英文，字符串尽量复杂，然后就会成功

方法区用于存放 Class 的相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。对于这些区域的测试，基本的思路是运行时产生大量的类区填满方法区，直到溢出。

下面代码通过借助 CGLib 直接操作字节码运行时产生了大量的动态类。值得特别注意的是，当前很多主流框架，如 Spring、Hibernate，在对类进行增强时，都会使用到 CGLib 这类字节码技术，增强的类越多，就需要越大的方法区来保护动态生成的 Class 可以加入内存.

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * VM args:-XX:PermSize=10M -XX:MaxPermSize=10M
 */
public class JavaMethodAreaOOM {
    public static void main(String[] args) {
        while (true){
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setCallback(new MethodInterceptor() {
                @Override
                public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                    return methodProxy.invokeSuper(objects,args);
                }
            });
            enhancer.create();
        }
    }
    static class OOMObject{

    }
}
```

**结果：**

```java
Caused by: java.lang.OutOfMemoryError: PermGen space
```



方法区溢出也是一种常见的内存溢出异常，一个类要被垃圾收集器回收掉，判定条件时比较苛刻的。在经常动态生成大量 Class 的应用中，需要特别注意类的回收状况。这类场景除了上面提到的程序使用了 CGLib 字节码增强和动态语言之外，常见的还有：大量 OSGi 的应用(即使是同一个类文件，被不同的加载器加载也会视为不同的类)等

### 本机直接内存溢出

DirectMemory 容器可通过 **-XX:MaxDirectMemorySize**指定，如果不指定，则默认与 Java 堆最大值(-Xmx指定)一样，下面代码越过了 DirectByteBuffer 类，直接通过**反射获取 Unsafe 实例**进行**内存分配**(Unsafe 类的 getUnsafe()方法限制了只有引导类加载器才会返回实例，也就是设计者希望只有 rt.jar 中的类才能使用 Unsafe 的功能)

> 虽然使用 DirectByteBuffer 分配内存也会抛出内存溢出异常，但它抛出异常时并没有真正向操作系统申请分配内存，而是通过计算得知内存无法分配，于是手动抛出异常，真正申请分配内存的方法是 **unsafe.allocateMemory()**

```java
import sun.misc.Unsafe;

import java.lang.reflect.Field;

/**
 * VM args:-Xmx20M -XX:MaxDirectMemorySize=10M
 */
public class DirectMemoryOOm {
    private static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws IllegalAccessException {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe) unsafeField.get(null);
        while(true){
            unsafe.allocateMemory(_1MB);
        }
    }
}
```

**结果：**

```java
Exception in thread "main" java.lang.OutOfMemoryError
```

由 DirectMemory 导致的内存溢出，一个明显的特征是在 Heap Dump 文件中不会看见明显的异常，如果读者发现 OOM 之后 Dump 文件很小，而程序中又直接或间接使用了 NIO ，那就可以考虑检查一下是不是这方面的原因。

# 小结

- 虚拟机的内存是如何划分的
- 哪部分区域、什么样的代码和操作可能导致内存溢出异常