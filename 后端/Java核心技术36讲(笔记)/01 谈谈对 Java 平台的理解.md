# Java 平台的理解
## 正文总结
> 谈谈对 Java 平台的理解?
1. 跨平台(引入下面的问题)
2. GC

----

> Java 是解释执行?
javac 编译成 `.class` 字节码文件,然后 JVM 内嵌的解释器讲字节码转换成最终的机器码。

**JVM**:屏蔽了`操作系统`和`硬件`的细节,大多数使用 Oracle JDK 提供的 Hotspot JVM,都提供了 JIT(Just-In-Time)编译器-动态编译器<br>
**JIT**：在运行时讲热点代码编译成机器码,这种情况下,部分热点代码(重复使用的代码,一般为方法内代码)就属于编译执行,而不是解释执行。

**编译**: AOT(Ahead-of-Time Compilation);
## 需要弄懂的

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0vhxjr6jtj31820m5aij.jpg)

## 他人精辟总结

****

1.  
```
JVM的内存模型,堆、栈、方法区;
字节码的跨平台性;
对象在JVM中的强引用,弱引用,软引用，虚引用,是否可用finalise方法救救它?
双亲委派进行类加载,什么是双亲呢?
双亲就是多亲, 一份文档由我加载,然后你也加载,这份文档在JVM中是一样的吗?
多态思想是Java需要最核心的概念,也是面向对象的行为的一个最好诠释;
理解方法重载与重写在内存中的执行流程,怎么定位到这个具体方法的。
```


 2.
 ```
发展流程,
JDK5(重写bug) ,
JDK6(商用最稳定版) ,
JDK7(switch的字符串支持) ,
JDK8(函数式编程) , -直在发展进化。
 ```


 3. 理解祖先类Object,它的行为是怎样与现实生活连接起来的。


 4.  理解23种设计模式,因为它是道与术的结合体。
