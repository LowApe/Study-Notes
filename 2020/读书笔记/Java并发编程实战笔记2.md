# 03 对象的共享

同步还有一个重要的方面：**内存可见性**(Memory Visibility)。当一个线程修改了对象状态后，其他线程能够**看到发生的状态变化**

## 可见性

```java
public class NoVisibility {
    private static boolean ready=false;
    private static int number=0;
    private static  class ReaderThread extends Thread{
        @Override
        public void run() {
            while(!ready)
                Thread.yield();
            System.out.println(number);
        }
    }
    public static void main(String[] args) throws InterruptedException {
        new ReaderThread().start();
        Thread.sleep(10000); // 我自己debug 使用的，在主线程的这 10s 一直在执行 ReaderThread 线程的while循环，
        // 直到 ready = true 输出 47
        number = 47;
        ready = true;
    }
}
```

虽然我在自己机器上运行都能保证输出，但是书上说可能输出0，或者持续循环，因为存**重排序**，写入 ready 的值，但却没有看到 number 的值(意思main 线程先进ready =true，然后是 number 赋值,可是主线程难道不是按照顺序执行的吗？)

> Thread.yeild() 将线程的运行状态转成就绪状态，翻译为线程让步.
>
> 在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行调整，在缺乏足够同步的多线程程序中，对内存操作的执行顺序判断，几乎无法得出正确的结论

### 失效的数据

非同步获取变量可能是一个失效值。

```java
@NotThreadSafe
public class MutableInteger{
    private int value;
    public int get(){return value;}
    public void set(int value){this.value = value;}
}
```

上面对象在线程调用的情况可能更容易出现失效值。



### 非原子的 64位操作

Java 内存模型要求，变量的读取操作和写入操作都必须是原子操作，但对于非 `volatile`类型 long 和 double 变量，JVM 允许将 64 位的读操作或写操作分解为两个 32 位，当读取一个非 `volatile` 类型的 long 变量时，如果对该变量的读操作和写操作在不同的线程中执行，那么很可能会读取到某个值的高 32 位和另一个地 32 位，因此，即使不考虑失效数据问题，再多线程使用可变的 long 和 double 变量也是不安全的，除非用 `volatile` 来声明，保证变量的原子性



### 加锁于可见性

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhly1gfpr0si97pj30qa0n040h.jpg)

所谓的可见性，即在同一锁下可变的变量的都是可见的。可以理解为什么访问某个共享且可变的变量时要求所有线程在同一个锁上同步

> 加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，**所有的读操作或者写操作的线程都必须在同一个锁上同步**

### volatile 变量

volatile 变量，用来确保将变量的更新操作通知到其他线程，volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取 volatile 类型的变量时总会返回最新写入的值。

在访问 volatile 变量的**不会执行加锁操作，因此也就不会使执行线程阻塞**，因此 volatile 变量是一种比 `sychronized` 关键字更轻量级的同步机制。

> 当线程 A 首先写入一个 volatile 变量并且线程 B 随后读取该变量时，在写入 volatile 变量之前对 A 可见的所有变量的值，在 B 读取了 volatile 变量后，对 B 也是可见的。从**内存可见性**，写入 volatile 变量相当于退出同步代码块，而读取 volatile 变量就相当于进入同步代码块。

volatile 变量的一种典型用法:

```java
volatile boolean asleep;
...
    while（!asleep）
    	countSomeSheep();
```

这里检查某个状态标记以判断是否退出循环。如果 asleep 不是 volatile ，但其他线程修改 asleep 变量，执行判断的线程是发现不了的。

> 加锁机制既可以确保可见性又可以确保原子性，而 volatile 变量只能确保可见性。

满足一下所有条件时，才应该使用 volatile 变量：

- 对变量的写入操作不依赖变量的当前值，或者确保只有单线程更新变量的值
- 该变量不会与其他状态变量一起纳入不变性条件中
- 在访问变量时不需要加锁

除了上面的例子，还不知道还怎么用