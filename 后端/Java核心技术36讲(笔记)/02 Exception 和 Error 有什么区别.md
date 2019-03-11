# Exception 和 Error 有什么区别

## 请对比 Exception 和 Error ,另外,运行时异常和一般异常有什么区别?

- Exception 和 Error 都是继承了 Throwable 类,在 Java 中只有 Throwable 类型的实例可以抛出(throw) 或者捕获(catch),它是异常处理机制的基本组成类型

- Exception 和 Error 体现了 Java 平台设计者对不同异常情况的分类。
    - Exception 是程序正常运行中,可以预料的意外情况,可能并且应该被捕获,进行相应处理.
    - Error 是指在正常情况下,不太可能出现的情况,绝大部分的 Error 都会导致程序(比如 JVM 自身)处于非正常的、不可恢复状态,所以不便于也不需要捕获,常见的比如 OutOfMemoryError 之类,都是 Error 的子类


- Exception 又分为可检查(checked)异常和不检查(unchecked)异常,
    - 可检查异常在源代码里必须显示地进行`捕获`处理,这是编译期检查的一部分。Error 是 Throwable 不是 Exception
    - 不检查异常就是所谓的运行时异常,类似 NullPointerException、ArrayIndexOutOfBoundsException 之类,通常是可以编码避免的逻辑错误,具体根据需要来判断是否需要捕获,并不会在编译期强制要求

## 理解 Throwabale、Excption、Error 的设计和分类

## 理解 Java 语言操作 Throwable 的元素和实践。掌握 try-catch-finally 块,throw、throws 关键字等。

`try-with-resources` `multiple catch`

```java
try (BufferedReader br = new BufferedReader(…);
 BufferedWriter writer = new BufferedWriter(…)) {// Try-with-resources
// do something
catch ( IOException | XEception e) {// Multiple catch
 // Handle it
}
```

- 尽量不要捕获类似 Exception 这样的通用异常,而是应该捕获特定异常

- 不要生吞(swallow)异常。忽略异常

- `Throw early, catch late` 原则


##  性能角度审视 Java 的一场处理机制

- `try-catch` 代码段会产生额外的性能开销,它往往会影响 JVM 对代码进行优化,所以建议仅捕获有必要的代码,尽量不要一个大 try 包住整段代码

- Java 每实例化一个 Exception ,都会对当时的栈进行快照,这是一个相对比较重的操作。如果发生非常频繁,这个开销可就不能被忽略了。

- "JAVA语言规范将
派生于Error类或RuntimeException类的所有异常称为未检查(unchecked)异常，所有其他的
异常称为已检查(checked)异常。
