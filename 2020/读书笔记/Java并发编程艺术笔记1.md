# 01 并发编程的挑战



## 上下文切换

通过代码展示，并发和顺序执行程序次数不同，执行耗时不同

```java
package test;

/**
 * @Classname: ConcurrencyTest
 * @Description:
 * @author: qujiacheng
 * @Date: 2020/6/8
 * Version: 1.0
 */
public class ConcurrencyTest {
    private static final long count = 100001;

    public static void main(String[] args) throws InterruptedException {
        concurrency();
        serial();
    }

    private static void concurrency() throws InterruptedException {
        long start = System.currentTimeMillis(); // 开始时间
        Thread thread = new Thread(new Runnable() {
            public void run() {
                int a = 0;
                for (long i = 0; i < count; i++) {
                    a+=5;
                }
            }
        });
        thread.start();

        int b =0;
        for (long i = 0; i < count; i++) {
            b--;
        }
        long time = System.currentTimeMillis() - start; // 并发时间
        thread.join(); // 主线程等待子线程终止
        System.out.println("concurrency :" + time + "ms,b="+b);
    }


    private static void serial(){
        long start = System.currentTimeMillis();
        int a = 0 ;
        for(long i = 0; i< count;i++){
            a+=5;
        }
        int b =0;
        for (long i = 0; i < count; i++) {
            b--;
        }
        long time = System.currentTimeMillis() - start; // 并发时间
        System.out.println("serial :" + time + "ms,b="+b);
    }
}

```

> 经过测试执行显示，执行循环的小于在百万次，速度会比并发累加的慢
>
> 原因：线程有**创建和上下文切换的开销**



linux 中通过 `vmstat`命令查看上下文切换次数与时长

```shell
vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0  95992  79748 450372    0    0    32    20    3    2  2  1 97  0  0
```

> CS 列表示上下文切换次数



### 如何减少上下文切换

方式有：

| 减少上下文切换 | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| 无锁并发编程   | 避免使用锁。如将数据的 ID 按照Hash 算法取模分段，不同的线程处理不同段的数据(不理解) |
| CAS 算法       | Java 中的 Atomic 包使用 CAS 算法来更新数据，而不需要加锁     |
| 使用最少线程   | 避免创建不需要的线程，造成大量线程都处于等待状态             |
| 协程           | 在单线程里实现多任务的调度                                   |

### 实战

在linux 上找一个启动 java 程序`ps -ef` 找到程序并查看端口号。

```shell
# jstack 命令 dump 到指定文件线程信息
jstack PID > xxx
# 打印相关信息 
# grep:过滤文件内容
# 
grep java.lang.Thread.State dump | awk '{print $2$3$4$5}' | sort | uniq -c
10 RUNNABLE
      4 TIMED_WAITING(onobjectmonitor)
      6 TIMED_WAITING(parking)
      1 TIMED_WAITING(sleeping)
      5 WAITING(onobjectmonitor)
     12 WAITING(parking)
```

> 其中 5 wating 表示应用服务器的工作线程在 await。这里表示5个应用服务器线程空闲着

通过修改应用服务器线程池配置信息减少系统上下文切换，因为书上举的例子是 JBOSS。目前自己使用的 tomcat 的应用服务器



## 死锁



避免死锁的几个常见方式

1. 避免一个线程同时获取多个锁
2. 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源
3. 尝试使用定时锁，使用 lock.tryLock(timeout) 来替代使用内部锁机制
4. 对于数据库锁，加锁与解锁必须在一个数据库连接里



## 资源限制

> 资源限制：在进行并发编程时，程序的执行速度受限于计算机的硬件资源和软件资源。
>
> 硬件限制：带宽的上传/下载速度、硬盘读写速度和 CPU 的处理速度
>
> 软件限制：数据库的连接数和 socket 连接数等



**如何解决资源限制问题？**

> 如果硬件资源限制，可以考虑使用**集群**并行执行程序
> 如果软件资源限制，可以考虑**资源池**将资源复用



# 02 Java并发机制的底层实现原理

## volatile 的应用

> volatile 是轻量级的 synchronized ，它在多处理器开发中保证了共享变量的**可见性**



### volatile 的定义与实现原理

> Java 编程语言允许线程访问共享变量，为了保障共享变量能被准确和一致地更新，线程应该确保通过**排他锁**单独获得这个变量。如果一个字段被声明成 volatile，**Java 线程内存模型**确保所以线程看到这个变量的值是一致的。



有 volatile 变量修饰的共享变量进行**写操作**会多出 Lock 的编译代码。Lock 前缀的指令在多核处理器下会引发：

1. 将当前处理器缓冲行的数据写回到系统内存。
2. 这个写回内存的操作会使在其他 CPU 里缓存了该内存地址的数据无效(每个处理器嗅探检查缓存是否过期)

### volatile 的使用优化

JDK 7 并发包 `LinkedTransferQueue`，它在使用 volatie 变量时，用一种**追加字节**的方式优化队列出队和入队的性能。

> 一个对象的引用占用 4 个字节，它追加了 15 个变量，凑够 64 字节，为什么是 64 字节，能提高并发编程效率？因为高速缓存行是 64 个字节宽，并且队列头节点和尾节点都不足 64 字节的话，处理器会将他们都读到同一个**高速缓冲行**，这样当其中一个处理器修改头节点时，会将**整个缓存行锁定**，那么在缓存一致性机制的作用下，会导致其他处理器不能访问自己的尾节点，**而队列的入队和出队则需要不停修改头节点和尾节点**，追加 64 字节的方式填满高速缓冲区的缓冲行，避免头节点和尾节点夹在同一缓冲行，使头尾同时被锁。

**两种场景不应该使用这种方式：**

1. 缓冲行非 64 字节宽的处理器
2. 共享变量不会被频繁地写