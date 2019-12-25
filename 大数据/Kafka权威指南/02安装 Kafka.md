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

默认情况下，每个日志目录只使用一个线程。因为这些线程只是在服务器启动和关闭时会用到，所以完全可以设置大量的线程来达到并行操作的目的。特别是对于包含大量分区的服务器来说，一旦崩溃，恢复使用并行操作会节省时间



所配置的数字对应的是 log.dirs 指定的单个日志目录。也就是说，如果 `num.recovery.threads.per.data.dir`设置为 8 ，并且 `log.dir` 指定了 3 个路径，那么总共需要 24 个线程

### 6. auto.create.topics.enable

默认情况下，Kafka 会在如下几种情况下自动创建主题：

- 当一个生产者开始往主题写入消息时；
- 当一个消费者开始从主题读取消息时
- 当任意一个客户端向主题发送元数据请求时

