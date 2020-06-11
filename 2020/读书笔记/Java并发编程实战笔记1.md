# 02 线程的安全性

- 编写线程安全的代码，其核心在于要对**状态访问操作**进行管理，特别是对**共享的和可变的**的状态的访问

- 一个对象是否需要是线程安全的，取决于它是否被多个线程访问

- Java 中主要同步机制是关键字 `synchroniazed`，它提供了一种独占的加锁方式，但同步这个术语，还包括`volatile`类型的变量，显示锁以及原子变量

- 当设计线程安全的类时，良好的面向对象技术、不可修改性，以及明晰的不可变性规范都能起到一定的帮助



## 什么是线程安全性

当多个线程访问某个类时，这个类始终都能表现出正确的行为，那就称这个类是线程安全的 

**示例：**

无状态的 Servlet ：它既不包含任何域，也不包含任何对其他类中域的引用。由于线程访问无状态对象的行为并不会影响其他线程中操作的正确性，因此无状态对象是线程安全的。大多数 Servlet 都是无状态的，从而极大地降低了在实现 Servlet 线程安全性时的复杂性。**只有当 Servlet 在处理请求时需要保存一些信息，线程安全性才会成为一个问题。**



## 原子性

**示例：**

在 Servlet 中添加 long 类型的域，每处理一个请求就将这个值加1

```java
@NotThreadSafe
public class UnsafeCountingFactorizer implements Servlet{
    private long count = 0;
    public long getCount(){return count;}
    
    public void service(ServletRequest req,ServletResponse resp){
         BigInteger i = extractFromRequest(req);
	     BigInteger[] factors = factor(i);
         ++count;
         encodeIntoResponse(resp,factors);
    }
}
```

> `++count`看上去只是一个操作，但这个操作**并非原子的**，实际上，它包含三个独立的操作：
>
> 1. 读取 count 的值
> 2. 将值 + 1
> 3. 将计算结果写入 count
>
> 这是一个读取-修改-写入的操作序列，并且其**结果状态**依赖于之前的状态

在多线程调用中可能出现时序执行交替而造成的数据完整性问题。在并发编程中，这种由于不恰当的执行时而出现不正确的结果称作：竞态条件

### 竞态条件

当某个计算的正确性取决于多个线程的交替执行时序时，那么就会发生竞态条件。要获得正确的结果，必须取决于事件的发生时序。最常见的竞态条件类型就是`先检查后执行`操作。比如判断创建实例。

```java
@NotThreadSafe
public class LazyInitRace{
	private Object instance = null;
    public Object getInstance(){
        if(instance == null){
            instance = new Object();
        }
        return instance;
    }
}
```

### 复合操作

要避免竞态条件，就必须在某个线程修改时，通过**某种方式**防止其他线程使用这个变量，从而确保其他线程只能在修改操作完成之前或之后读取和修改状态。

> 原子操作：对于访问同一个状态的所有操作，这个操作是一个以原子方式执行的操作

如果之前的 `++count`是原子操作，那么竞态条件就不会发生，我们将读取-修改-写入或先检查后执行的操作统称为**复合操作**，我们将复合操作以**原子方式**执行来确保线程的安全性。

```java
@NotThreadSafe
public class UnsafeCountingFactorizer implements Servlet{
    private final AtomicLong count = new AtomicLong(0);
    public long getCount(){return count;}
    
    public void service(ServletRequest req,ServletResponse resp){
         BigInteger i = extractFromRequest(req);
	     BigInteger[] factors = factor(i);
         count.increamentAndGet();
         encodeIntoResponse(resp,factors);
    }
}
```

此种方式是通过使用线程安全类来保障原子性，还有一种方式是加锁。

## 加锁机制

并不是使用原子引用就能保证线程安全的，比如下面的例子：

```java
@NotThreadSafe
public class UnsafeCountingFactorizer implements Servlet{
    private final AtomicReference<BigInteger> lastNumber = new AtomicReference<>();
    private final AtomicReference<BigInteger[]> lastFactors = new AtomicReference<>();
    
    public void service(ServletRequest req,ServletResponse resp){
         BigInteger i = extractFromRequest(req);
        if(i.equals(lastNimber.get())){
            encodeIntoResponse(resp,lastFactors resp);
        }else{
            BigInteger[] factors = factor(i);
            lastNumber.set(i);
            lastFactors.set(factors);
            encodeIntoResponse(resp,factors)
        }
    }
}
```

尽管这些原子引用本身都是线程安全的，但在这里存在着竞态条件，这可能产生错误的结果。尽管对 set 方法的每次调用都是原子的，但仍然无法**同时更新 lastNumber 和 lastFactors**。如果只修改了其中一个变量，那么在这两次修改操作之间，其他线程将发现**不变性条件被破坏了**



> 要保持状态的一致性，就需要在单个原子操作中更新所有相关的状态变量。
>
> AtomicLog 是一种代替 long 类型整数的线程安全类，类似，AtomicReference 是一种替代**对象引用**的线程安全类

### 内置锁

Java提供内置的**锁机制来支持原子性：**同步代码块(Synchronized Block)：

1. 锁的对象引用
2. 锁保护的代码块

> 以关键字 synchronized 来**修饰的方法**是一种横跨整个方法体的同步代码块，其中该同步代码块的**锁就是方法调用所在的对象**

```java
synchronized (lock){
    // 访问或修改由锁保护的共享状态
}
```

线程在进入同步代码块之前会自动获得锁，在退出同步代码块时自动释放锁，无论是正常控制路径退出，还是代码块抛出异常退出，都自动释放锁。这种方式获得锁的唯一途径就是进入锁保护的同步代码块或方法。

因为每次只有一个线程执行内置锁保护的代码块，因此，有这个锁保护的同步代码块会以原子方式执行。**并发环境中的原子性与事务应用程序的原子性有着相同的含义**。针对上面的 Servlet 代码，向 service 方法添加`synchronized` 关键字可以确保线程安全。

### 重入

当某个线程请求一个由其他线程持有锁时，该线程会阻塞。然而，内置锁是可重入的，如果某线程获得一个已经持有的锁，那么请求就会成功(为什么还要获得一次？)，**重入**意味着获取锁的操作的粒度是**线程**，而不是**调用**。

> 重入的实现是每个锁关联一个**计数器**和一个**所有者线程**。当计数器为 0 时，这个锁就被认为是没有被任何线程持有。如果一个线程请求未被持有的锁时，计数器 +1，所有者几下这个线程，如果**同一个线程**再次请求这个锁，计数值将递增，而当线程退出同步代码块时，计数器会递减，为 0 则这个锁被释放

重入避免了子类同步方法调用父类同步方法时，如果不重入会造成父类死锁(因为第一个线程从子类同步方法调用父类时，父类的锁已经给次线程了)

```java
public class Widget{
    public synchronized void doSomething(){
        ... // 对象锁是 Wigget
    }
}
public class LoggingWidget extends Widget(){
    public synchronized void doSomething(){
        // 对象锁是 LoggingWidget
		System.out.println(toString() + ":calling doSomeing");
        super.doSomething();
    }
}
```

每个 doSomething 方法在执行前都会获得 Widget 的锁（为什么？子类不是只有调用了才获取吗？）

自我理解：因为内置锁是可重入的，子类的同步方法拥有自己以及父类的锁，当再一次调用 `doSomething`还是持有父类的锁，所以不会发生死锁。





