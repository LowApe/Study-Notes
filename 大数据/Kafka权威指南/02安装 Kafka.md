# 安装 Kafka

- 介绍如何安装和运行 Kafka

- 包括如何设置 Zookeeper(Kafka 使用 Zookeeper 保存 Broker 的元数据)
- Kafka 的基本配置
- 如何为 Kafka 选择合适的硬件
- 如何在一个集群中安装多个 Kafka broker



# 环境搭建

## 安装 Java

需要先安装 Java 环境，推荐安装 Java 8

## 安装 Zookeeper

Kafka 使用 Zookeeper 保存集群的元数据信息和消息者信息。Kafka 发行版自带了 Zookeeper ，可以直接从脚本启动，不过安装一个完整版的 Zookeeper 也并不费劲

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1ga8nyj1fl8j30oq08c75l.jpg)

**Zookeeper 的 3.4.6** 稳定版已经在 Kafka 上做过全面测试

#### 1. 单机服务

下面演示如何使用基本的配置安装 Zookeeper，安装目录 `/usr/local/zookeepers`,数据目录 `var/lib/zookeeper`

```shell
#tar -zxf zookeeper-3.4.6.tar.gz
#mv zookeeper-3.4.6 /usr/local/zookeeper
#mkdir -p /var/lib/zookeeper
#cat > /usr/local/zookeeper/conf/zoo.cfg << EOF
> tickTime = 2000
> dataDir =/var/lib/zookeeper
> clientPort = 2181
> EOF
# /usr/local/zookeeper/bin/zkServer.sh start

```

现在可以连到 Zookeeper 端口上，通过发送四字命令 `srvr` 来验证 Zookeeper 是否安装正确

```shell

# telnet localhost 2181
Trying ::1...
Connected to localhost.
Escape character is '^]'.
srvr
Zookeeper version: 3.4.6-1569965, built on 02/20/2014 09:09 GMT
Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: standalone
Node count: 4
Connection closed by foreign host.


```

> ⚠️注意：如果启动不起来，查看zookeeper.out 文件显示报错信息



### 2. Zookeeper 群组(Ensemble)

Zookeeper 集群被称为群组。Zookeeper 使用的是一致性协议。所以建议每个群组里应该包含奇数个节点(比如 3 个、5个等)只有当群组里的大多数节点处于可用状态，Zookeeper 才能处理外部的请求。也就说，如果你有一个包含 3 个节点的群组，那么它允许一个节点失效。如果群组包含 5 个节点，那么它允许 2 个节点失效

> ⚠️注意：假设有一个包含 5 个节点的群组，如果要对群组做一些包含更换节点在内的配置更改，需要依次重启每一个节点。如果你的群租无法容忍多个节点失效，那么在进行群组维护时就会存在风险。不过，也不建议一个群组包含超过**7个节点**，因为 Zookeeper 使用了一致性协议，节点过多会降低整个群组的性能。

群组需要有一些公共配置，上面列出了所有服务器的清单，并且**每个服务器**还要在数据目录中创建一个 **myid 文件**, 用于指明自己的 ID。如果群组服务器的机器名是 `zoo1.example.com`、`zoo2.example.com、zoo3.example.com`，那么配置文件可能是这样的:

```properties
tickTime = 2000
dataDir = /var/lib/zookeeper
initLimit = 20
syncLimit = 5
server.1 = zoo1.example.com:2888:3888
server.2 = zoo1.example.com:2888:3888
server.3 = zoo1.example.com:2888:3888
```

在这个配置中，`initLimit`表示用于在从节点与主节点之间建立初始化连接的时间上限，`syncLimit`表示允许从节点与主节点处于不同步状态的时间上限。这两个值都是 tickTime 的倍数，所以 initLimit 是 `20*2000ms`，也就是 40s 。配置里还列出了群组中所有服务器的地址。`server.X=hostname:peerPort:leaderPort`

| 名称       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| X          | 服务器的 ID，它必须是一个整数，不过不一定要从 0 开始，也不要求是连续的 |
| hostname   | 服务器的机器名或 IP 地址                                     |
| peerPort   | 用于节点间通信的 TCP 端口                                    |
| leaderPort | 用于首领选举的 TCP 端口                                      |

>  客户端只需要通过 clientPort 就能连接到群组，而群组节点间的通信则需要同时用到 3 个端口(peerPort、leaderPort、clientPort)

除了公共的配置文件外，每个服务器都必须在 dataDir 目录中创建一个叫做 myid 的文件，文件里要包含

- 服务器 ID(公共配置 ID 一致)

完成后，彼此间就可以通行了



# 安装 Kafka Broker

配置好 Java 和 Zookeeper 之后，接下来就可以安装 Kafka 了。

Zookeeper 配置日志输出

```shell
DataLogDir = /tmp/kafka-logs
```

操作

```shell
# tar -zxf kafka_2.11-2.4.0.1.tgz
# mv kafka_2.11-2.4.0 /usr/local/kafka
# mkdir /tmp/kafka-logs
# sudo bin/kafka-server-start.sh -daemon /usr/local/kafka/kafka_2.11-2.4.0/config/server.properties 

```

创建完毕，就可以对这个集群做一些简单的操作来验证是否安装正确：

```shell
# /usr/local/kafka/kafka_2.11-2.4.0/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
Created topic "test"
# /usr/local/kafka/kafka_2.11-2.4.0/bin/kafka-topics.sh --zookeeper localhost:2181 --describe  --topic test
Topic: test	PartitionCount: 1	ReplicationFactor: 1	Configs: 
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0

```

往测试主题上发布消息

```shell
# bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
>send a Test message 1
>send a Test message 2

```

从测试主题上读取消息：

```shell
# bin/kafka-console-consumer.sh --bootstrap-server  localhost:9092 --topic test --from-beginning
send a Test message 1
send a Test message 2

```

# Broker 配置

Kafka 发行包里自带的配置样本可以用来安装单机服务，但并不能满足大多数安装场景的要求。Kafka有很多配置选项，涉及安装和调优的方方面面/不过大多数调优选项可以使用默认设置，除非你对调优有特别的要求

## 常规配置

有些参数是单个服务器最基本的配置，它们中的大部分需要经过修改后才能用在集群里

### 1.broker.id

每个 broker 都需要有一个表示符，使用 broker.id 来表示。它的默认值是 0。

> - 这个值在整个 Kafka 集群里必须是唯一的
> - 建议把它们设置成与机器名具有相关性的整数，这样在进行维护时，将 ID 号映射到机器名就不那么麻烦
> - 例如🌰：如果机器名包含唯一数字(host1.example.com、host2.examole.com)，那么用这些数字来设置 broker.id 

### 2. port

如果使用配置样本来启动 Kafka，它会监听 9092 。修改 port 配置参数设置任意端口

> 不建议使用 1024 以下的端口

### 3. zookeeper.connect

用于保存 broker 元数据的 Zookeeper 地址是通过 zookeeper.connect 来指定的。 `localhost:2181`表示这个 Zookeeper 是运行在本地的 2181 端口上。一般使用 `hostname:port/path`列表

- hostname 是 Zookeeper 服务器的机器名或 IP地址
- port 是 Zookeeper 的客户端连接端口
- /path 是可选的 Zookeeper 路径，作为 Kafka 集群的 `chroot`🤔环境

> ⚠️注意：为什么使用 chroot 路径，在 Kafka 集群里使用 chroot 路径是一种最佳实践。Zookeeper群组可以共享给其他应用程序，即使还有其他 Kafka 集群存在，也不会产生冲突。最好是在配置文件里**指定一组** Zookeeper 服务器，用**分号**把它们隔开。一旦有一个 Zookeeper 服务器宕机，broker 可以连接到 Zookeeper 群组的另一个节点

### 4. log.dirs

kafka 把所有消息都保存在磁盘上，存放这些日志片段的目录是通过 `log.dirs`指定的。它是一组用逗号分隔的本地文件系统路径。如果制定了多个目录会根据"最少使用"原则，把同一个分区的日志片段保存到同一个路径下。要注意，broker 会拥有**最少数目分区的路径**新增分区，而不是往拥有最小磁盘空间的路径新增分区

### 5. num.recovery.threads.per.data.dir

对于如下 3 种情况，Kafka 会使用可配置的线程池来处理**日志片段**

- 服务器正常启动，用于打开每个分区的日志片段
- 服务奔溃后重启，用于检查和截短每个分区的日志片段
- 服务器正常关闭，用于关闭日志片段

默认情况下，每个日志目录只使用一个线程。因为这些线程只是在服务器启动和关闭时会用到，所以完全可以设置大量的线程来达到并行操作的目的。特别是对于包含大量分区的服务器来说，一旦崩溃，恢复使用并行操作节省时间

所配置的数字对应的是 log.dirs 指定的单个日志目录。也就是说，如果 `num.recovery.threads.per.data.dir`设置为 8 ，并且 `log.dir` 指定了 3 个路径，那么总共需要 24 个线程

### 6. auto.create.topics.enable

默认情况下，Kafka 会在如下几种情况下自动创建主题：

- 当一个生产者开始往主题写入消息时；
- 当一个消费者开始从主题读取消息时
- 当任意一个客户端向主题发送元数据请求时

## 主题的默认配置

Kafka 为新创建的主题提供了很多默认配置参数。可通过管理工具，为每个主题单独配置一部分参数，比如分区个数和数据保留策略。服务器的默认配置可以作为基准，它们适用于大部分主题。

### 1. num.partitions

指定新建主题将包含多少分区，默认 1.

> ⚠️注意：只能增加主题分区个数，如果要减少分区的个数，需要手动创建该主题

**如何选定分区数量**

为主题选定分区数量并不是一件可有可无的事情，在进行数量选择时，需要考虑如下几个因素

- 主题需要到达多大的吞吐量？
- 从单个分区读取数据的最大吞吐量是多少？
- 每个 broker 包含的分区个数、可用的磁盘空间和网络带宽
- 如果消息是按照不同的键来写入分区的，那么为已有的主题新增分区就会很困难
- 单个 broker 对分区个数是有限制的，因为分区越多，占用的内存越多，完成首领选举需要的时间也越长

> 如果你估算出主题的吞吐量和消费者吞吐量，可以用主题吞吐量除以消费者吞吐量算出分区的个数

### 2. log.retention.ms

Kafka 通常根据时间来决定数据保留时间。默认`log.retention.hours`参数设置时间，默认值为 168，也就是一周。还有两个`ms`和`minutes`决定消息多久以后会被删除

> 根据时间保留数据是通过检查磁盘上**日志片段文件**的最后修改时间来实现的

### 3. log.retention.bytes

另一种通过保留的消息字节数来判断消息是否过期。它的值通过参数 `log.retention.bytes`来指定，作用在每一个分区上.超过部分会被删除

> 如果字节大小和时间保留数据同时存在，则满足任意一个条件就会删除消息

### 4. log.segment.bytes

以上的设置都作用在**日志片段**上，而不是作用在单个消息上。当消息到达 broker 时，它门被追加到分区的当前日志片段上。当日志片段到达`log.segment.bytes`指定的上限时，当前日志片段就会关闭，新的日志片段被打开。如果一个日志片段被关闭，就开始等待过期。这个参数的值越小，就会频繁关闭与分配新文件，**从而降低磁盘写入的整体效率**

🌰例如：一个主题每天只接收 100MB 的消息，而 `log.segement.bytes`使用默认设置，那么需要 10 天时间才能填满一个日志片段。**日志片段被关闭之前消息是不会过期的**，所以如果 `log.retention.ms` 被设为 一周,那么日志片段最多需要 17 天 才会过期。(10+7,+7是因为日志片段最后一天加入的消息要保留7天)

> 使用时间戳获取偏移量：日志片段的大小会影响使用时间戳获取偏移量。在使用时间戳获取日志偏移量时，Kafka 会检查分区里最后修改时间大于指定时间戳(已经关闭的)，该日志片段的前一个文件的最后修改时间小于指定时间戳。然后，Kafka 返回该日志片段(也就是文件名)开头的偏移量。对于使用时间戳获取偏移量的操作来说，日志片段越小，结果越准确🤔？？？

### 5. log.segment.ms

另一个可以控制日志片段关闭时间的参数是 `log.segment.ms`,它指定了多长时间之后日志片段会被关闭。就像 `log.retention.bytes`和 `log.retention.ms`这两个参数一样,`log.segement.bytes` 和`log.retention.ms`这两个参数之间也不存在互斥问题。日志片段会在大小或时间达到上限时被关闭。

### 6. message.max.bytes

broker 通过设置 `message.max.bytes`参数来限制单个消息的大小,默认 1MB。这里是指压缩消息，也就是说生产者尝试发送的消息可以大于这个值，压缩后的值要小于这个值

> ⚠️注意：这个值对性能有显著的影响。值越大，那么负责处理网络连接和请求的线程就需要花越多的时间来处理这些请求。它还会增加磁盘写入块的大小，从而影响 IO 吞吐量

> 💡提示：消费者客户端设置的 `fetch.message.max.bytes`必须与服务器设置的消息大小进行**协调**。如果这个值比 `message.max.bytes`小，那么消费者就无法读取比较大的消息，导致出现消费者被阻塞的情况。在为集群里的 broker 配置 `replica.fetch.max.bytes`参数时，也遵循同样的原则

# 硬件的选择

如果比较关注性能，就需要考虑几个会影响整体性能的因素：

- 磁盘吞吐量和容量
- 内存
- 网络
- CPU

在确定了性能关注点之后，就可以在预算范围内选择最优化的硬件配置

## 磁盘吞吐量

- HDD，使用磁盘列阵技术
- SDD

## 磁盘容量



## 内存

服务器的内存容量是影响客户端性能的主要因素。磁盘性能影响生产者，而**内存影响消费者**。运行 Kafka 的 JVM 不需要太大的内存，剩余的系统内存可以用作页面缓存，或者用来缓存正在使用中的日志片段。这也就是为什么不建议把 Kafka 同其他重要的应用程序部署在一起的原因，它们需要共享页面缓存，最终会降低 Kafka 消费者的性能



## 网络

**网络吞吐量**决定了 Kafka 能够处理的最大数据流量。它和**磁盘存储**制约 Kafka 扩展规模的因素。

- 其他的操作，如集群复制和镜像也会占用网络流量

## CPU

Kafka 对计算处理能力的要求相对较低，不过它在一定程度上还是会影响整体的性能。客户端为了优化网络和磁盘空间，会对消息进行压缩。服务器需要对消息进行批量解压，设置偏移量，然后重新进行批量压缩，再保存到磁盘上。这就是 Kafka 对计算处理能力有所要求的地方

# 云端的Kafka

Kafka 一般被安装在云端，比如 AWS。Kafka 实例考虑因素

- 保留数据的大小开始考虑
- 考虑生产者方面的性能
- 考虑低延迟，那么就需要专门为 I/O 优化过的使用固态硬盘的实例，否则，使用配备了临时存储的实例就可以了
- 然后选择 CPU 和 内存

> 使用 AWS，一般会选择 m4 实例或 r3 实例。m4 实例运行较长时间地保留数据，不过磁盘吞吐量会小一些，因为它使用的是弹性块存储。r3则使用固态硬盘，具有较高的吞吐量，但保留的数据量会有所限制。



# Kafka 集群

- 单个 Kafka 服务器足以满足本地开发或 POC 要求
- 使用集群最大的好处是可以跨服务器进行**负载均衡**，使用复制功能避免因**单点故障**造成的数据丢失
- 集群确保客户端提供高可用性

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1ga9vmbrlemj30si0fg406.jpg)

## 需要多少个 Broker

- 需要的保留数据除以每个 broker 存储的容量，如果启动了数据复制，那么至少还需要一倍空间，取决于配置的复制系数是多少
- 集群处理请求的能力

## broker 配置

- 添加一个 broker 到集群里，只需要修改两个配置参数。首先，所有 broker 都必须配置相同的 `zookeeper.connect` ，该参数指定了用于**保存元数据的 Zookeeper 群组和路径**
- 每个 broker 必须为 `broker.id`参数设置唯一的值

## 操作系统调优

大部分 Linux 发行版默认的内核调优参数设置已经能够满足大多数应用程序的运行需求，不过还是可以通过调整一些参数来进一步提升 Kafka 的性能。这些参数主要与**虚拟内存**、**网络子系统**和用来存储日志片段的**磁盘挂载点**有关.这些参数一般配置在 `etc/sysctl.conf`文件里，不过对内核参数进行调整时，最好参考操作系统的文档



### 虚拟内存

- 尽量避免内存交换。如果虚拟内存被交换到磁盘，说明没有多余内存可以分配给页面缓存了
- 一种避免内存交换的方法是不设置任何交换分区。建议 `vm.swappiness`参数的值设置得小一点，比如 1.该参数指明了虚拟机的子系统将如何使用交换分区，而不是只把内存页从页面缓存里移除
- 脏页会被写入到磁盘上，调整内存对脏页的处理方式可以让我们从中获益。可以通过将 `vm.dirty_background_ratio`设置小于 10(指系统内存的百分比)，但不能为 0，因为设置为 0则不提供缓冲的能力

> 脏页－linux[内核](https://baike.baidu.com/item/内核)中的概念，因为[硬盘](https://baike.baidu.com/item/硬盘)的读写速度远赶不上内存的速度，系统就把读写比较频繁的数据事先放到[内存](https://baike.baidu.com/item/内存)中，以提高读写速度，这就叫高速缓存，linux是以页作为[高速缓存](https://baike.baidu.com/item/高速缓存)的单位，当进程修改了高速缓存里的数据时，该页就被内核标记为脏页，内核将会在合适的时间把脏页的数据写到磁盘中去，以保持高速缓存中的数据和磁盘中的数据是一致的。---百度百科

- 设置`vm.dirty_ratio`参数可以增加被内核进程刷新到磁盘之前到脏页数量(60～80)，如果设置较高的值,建议启用 Kafka 的复制功能，避免因系统崩溃造成数据丢失
- 可以在 `/proc/vmstat` 文件里查看当前脏页数量

```shell
# cat /proc/vmstat | egrep "dirty|writeback"
```

### 2. 磁盘



### 3. 网络



# 生产环境的注意事项

## 垃圾回收器选项

为应用程序调整 Java 垃圾回收参数。Java 7 带来了 G1 垃圾回收器。在应用程序的整个生命周期，G1 会自动根据工作负载情况进行自我调节，而且它的停顿时间是恒定的。**它可以轻松地处理大块的堆内存，把对内存分为若干小块区域，每次停顿时并不会对整个堆空间进行回收**。正常情况只需要很少的配置能完成这些工作

- MaxGCPauseMillis 每次GC回收停顿的时间
- InitiatingHeapOcupancyPercent 该参数指定了在 G1 启动垃圾回收之前可以使用的堆内存百分比。这个百分比包括新生代和老生代的内存

> 如果一台服务器有 64GB 内存，使用 5GB 堆内存来运行 Kafka，那么可以参考以下的配置：MaxGCPauseMillis 可以设为 20ms; InitiatingHeapOccupancyPercent 可以设为 35，这样可以让垃圾回收比默认的要早一些启动

Kafka 的启动脚本并没有启动 G1回收器，而是使用了 Parallel New 和 CMS (Concurrent Mark-Sweep，并发标记和清除)垃圾回收器，可以通过**环境变量来修改**

```shell
# export JAVA_HOME=
# export KAFKA_JVM_PERFORMANCE_OPTS="-server 
-XX:+UseG1GC"
-XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOcupancyPercent=35
# /usr/local/kafka/bin/kafka-server-start.sh -daemon
#
```

## 数据中心布局

在开发阶段，人们并不会太关心 Kafka 服务器在数据中心所处的物理位置。

- 最好把集群的 broker 安装在不同的**机架**上，至少不要让它们共享可能出现单点故障的基础设施，比如电源和网络

## 共享 Zookeeper

Kafka 使用 Zookeeper 来保存 broker，主题和分区的元数据信息。对于一个包含多个节点的 Zookeeper 群组来说，Kafka 集群的这些流量并不算多，那些写操作只是用于构建消费者群组或集群本身。

> Kafka 消费者和 Zookeeper 之间还是有一个值的注意的地方，之前消息和分区偏移量放置到 Zookeeper 中，可以存在必要的时间间隔。不过还是让消费者把偏移量提交到 Kafka 服务器上，消除对 Zookeeper 的依赖

- Kafka 对 Zookeeper 的延迟和超时比较敏感，与 Zookeeper 群组之间的一个通信异常就可能导致 Kafka 服务器出现无法预测的行为



