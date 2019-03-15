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
- 将 class 自身声明为 final，这样别人就不能扩展来绕过限制了。
- 将所有成员变量定义为 private 和 final，并且不要实现 setter 方法。
- 通常构造对象时，成员变量使用`深度拷贝`来初始化，而不是直接赋值，这是一种防御措施，因为你无法确定输入对象不被其他人修改。
- 如果确实需要实现 getter 方法，或者其他可能会返回内部状态的方法，使用 copy-on-write原则，创建私有的 copy。
