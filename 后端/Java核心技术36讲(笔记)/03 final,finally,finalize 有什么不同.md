# final,finally,finalize 有什么不同
final 可以修饰类,方法,变量,分别有不同的意义,final 修饰的 class 不可以被继承
final 的变量是不可以修改的,而final的方法不可以重写

finally 则是 Java 保证重点代码一定要被执行的一种机制.我们可以使用 try-finally 或者 try-catch-finally 来进行类似关闭 JDBC 连接,保证 unlock 锁等动作

finalize 是基础类 java.lang.Object 的一个方法,它的设计目的是保证对象在被垃圾收集前完成特定资源的回收.finalize 机制现在已经不推荐使用了 JDK 9 开始被标记为 deprecated

## final

- 使用 final 修饰参数或者变量，也可以清楚地避免意外赋值导致的编程错误，甚至，有人明确推荐将所有方法参数、本地变量、成员变量声明成 final。

- final 变量产生了某种程度的不可变（immutable）的效果，所以，可以用于保护只读数据，尤其是在并发编程中，因为明确地不能再赋值 final 变量，有利于减少额外的同步开销，也可以省去一些防御性拷贝的必要。

```java
final List<String> strList = new ArrayList<>();
strList.add("Hello");
strList.add("world");
List<String> unmodifiableStrList = List.of("hello", "world");
unmodifiableStrList.add("again");
```

final 只能约束 strList 这个引用不可以被赋值，但是 strList 对象行为不被 final 影响，添加元素等操作是完全正常的。如果我们真的希望对象本身是不可变的，那么需要相应的类支持不可变的行为。在上面这个例子中，List.of 方法创建的本身就是不可变 List，最后那句 add 是会在运行
时抛出异常的。

-----
## finally
```java
try {
// do something
System.exit(1);
} finally{
System.out.println(“Print from finally”);
}
```
> 上面不会执行


## fianlize

简单说，你无法保证 finalize 什么时候执行，执行的是否符合预期。使用不当会影响
性能，导致程序死锁、挂起等。


****

1. 注意，final 不是 immutable！

Immutable 在很多场景是非常棒的选择，某种意义上说，Java 语言目前并没有原生的不可变支持，如果要实现 immutable 的类，我们需要做到：
- 将 ` class` 自身声明为 `final`，这样别人就不能扩展来绕过限制了。
- 将所有成员变量定义为` private` 和 `final`，并且不要实现 `setter` 方法。
- 通常构造对象时，成员变量使用`深度拷贝`来初始化，而不是直接赋值，这是一种防御措施，因为你无法确定输入对象不被其他人修改。
- 如果确实需要实现 `getter` 方法，或者其他可能会返回内部状态的方法，使用 `copy-on-write`原则，创建私有的 `copy`。


2. finalize 真的那么不堪？

finalize 的执行是和垃圾收集关联在一起的，一旦实现了非空的 finalize 方法，就会导致相应对象回收呈现数量级上的变慢，有人专门做过 benchmark，大概是 `40~50` 倍的下降。

finalize 被设计成在对象被垃圾收集前调用，这就意味着实现了 finalize 方法的对象是个“特殊公民”，JVM 要对它进行额外处理。finalize 本质上成为了快速回收的阻碍者，可能导致你的对象经过多个垃圾收集周期才能被回收。

有人也许会问，我用 System.runFinalization () 告诉 JVM 积极一点，是不是就可以了？也许有点用，但是问题在于，这还是不可预测、不能保证的，所以本质上还是不能指望。实践中，因为finalize 拖慢垃圾收集，导致大量对象堆积，也是一种典型的导致 OOM 的原因。


从另一个角度，我们要确保回收资源就是因为资源都是有限的，垃圾收集时间的不可预测，可能会极大加剧资源占用。这意味着对于消耗非常高频的资源，千万不要指望 finalize 去承担资源释放的主要职责，最多让 finalize 作为最后的“守门员”，况且它已经暴露了如此多的问题。这也是为什么我推荐，资源用完即显式释放，或者利用资源池来尽量重用

3. 有什么机制可以替换 finalize 吗？

Java 平台目前在逐步使用 java.lang.ref.Cleaner 来替换掉原有的 finalize 实现。Cleaner 的实现利用了幻象引用（PhantomReference），这是一种常见的所谓 post-mortem 清理机制,利用幻象引用和引用队列，我们可以保证对象被
彻底销毁前做一些类似资源回收的工作，比如关闭文件描述符（操作系统有限的资源），它比finalize 更加轻量、更加可靠。

吸取了 finalize 里的教训，每个 Cleaner 的操作都是独立的，它有自己的运行线程，所以可以避免意外死锁等问题。


实践中，我们可以为自己的模块构建一个 Cleaner，然后实现相应的清理逻辑。下面是 JDK 自身提供的样例程序：

```java
public class CleaningExample implements AutoCloseable {
// A cleaner, preferably one shared within a library
private static final Cleaner cleaner = <cleaner>;
static class State implements Runnable {
State(...) {
// initialize State needed for cleaning action
}
public void run() {
// cleanup action accessing State, executed at most once
}
}
private final State;
private final Cleaner.Cleanable cleanable
public CleaningExample() {
this.state = new State(...);
this.cleanable = cleaner.register(this, state);
}
public void close() {
cleanable.clean();
}
}
```

注意，从可预测性的角度来判断，Cleaner 或者幻象引用改善的程度仍然是有限的，如果由于种种原因导致幻象引用堆积，同样会出现问题。所以，Cleaner 适合作为一种最后的保证手段，而不是完全依赖 Cleaner 进行资源回收，不然我们就要再做一遍 finalize 的噩梦了。
