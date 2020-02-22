# RocketMQ 安装

前期准备

- [安装 Java]()
- [安装 Maven]()

> ⚠️ 跳转到写的安装文章

## 1.下载源码

[点击下载4.6.1](https://archive.apache.org/dist/rocketmq/4.6.1/rocketmq-all-4.6.1-source-release.zip)

```shell
$ unzip rocketmq-all-4.6.1-source-release.zip # 解压 zip 包
$ cd rocketmq-all-4.6.1-source-release # 进入目录
```

## 2.安装与启动

```shell
$ mvn -Prelease-all -DskipTests clean install -U # 使用 maven 编译,生成 target 编译后目录文件
$ cd distribution/target/rocketmq-4.6.1/rocketmq-4.6.1/ # 进入目录
$ ls # 
LICENSE		README.md	bin		lib
NOTICE		benchmark	conf		nohup.out
$ nohup sh bin/mqnamesrv & # 启动 namesrv ,文件 nohup.out 查看日志
$ nohup sh bin/mqbroker -n localhost:9876 & # 启动 broker
```

## 3.消息测试

```shell
$ export NAMESRV_ADDR=localhost:9876 # 设定 namesrv 地址
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer # 启动生产者
...
20:35:52.148 [NettyClientSelector_1] INFO  RocketmqRemoting - closeChannel: close the connection to remote address[192.168.0.105:10911] result: true
20:35:52.148 [NettyClientSelector_1] INFO  RocketmqRemoting - closeChannel: close the connection to remote address[127.0.0.1:9876] result: true
$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer # 启动消费者
...
ConsumeMessageThread_5 Receive New Messages: [MessageExt [queueId=1, storeSize=227, queueOffset=491, sysFlag=0, bornTimestamp=1582374952117, bornHost=/192.168.0.105:56988, storeTimestamp=1582374952117, storeHost=/192.168.0.105:10911, msgId=C0A8006900002A9F000000000006CD8B, commitLogOffset=445835, bodyCRC=673705983, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=500, CONSUME_START_TIME=1582375026899, UNIQ_KEY=FE80000000000000AB4EA085AE3E05AB00000D716361709104B503C5, CLUSTER=DefaultCluster, WAIT=true, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 57, 54, 53], transactionId='null'}]]
```

## 4. 关闭服务

```shell
$ sh bin/mqshutdown broker # 关闭 broker
$ sh bin/mqshutdown namesrv  # 关闭 namesrv
```



# 生产环境下的配置和使用

为了消除单点故障，增加可靠性或增大吞吐量，可以在多台机器上部署多个 `NameServer`和`Broker`,为每个 `Broker` 部署一个或多个 `Slave`

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1gc5i1cr4boj314e0hojvt.jpg)

## 1.多机部署

> 部署一个 双主、双从的 RoketMQ 集群，每台机器启动一个 Master 角色和 

示例配置在 `conf/2m-2s-sync` 文件夹下

- 2m-2s 双主、双从
- sync 同步

准备两个**物理机**,分别启动 nameSrv

## 2.配置参数

物理机 `1-master`配置 `broker-a.properties`:

```properties
# nameServer 地址可以用分号配置多个
namesrvAddr=ip1:9876;ip2:9876
# 集群机器如果多，可以通过名称划分多个 Cluster ，每个 Cluster 表示一个业务群
brokerClusterName=DefaultCluster
# Master 和 Salve 使用相同的名称表明相互关系
brokerName=broker-a
# 0-表示 Master 1-表示 SLAVE
brokerId=0
# 凌晨 4点 做删除操作
deleteWhen=04
# 在磁盘保存消息的时间 48小时
fileReservedTime=48
# 同步的 master
brokerRole=SYNC_MASTER  
# 磁盘刷新策略，分异步刷新和同步刷新
flushDiskType=ASYNC_FLUSH
# broker 监听端口
listenPort=10911
# 存储消息以及信息的根目录
storePathRootDir=/Users/mac/Documents/AllMyDocuments/RocketStore
```

物理机 `1-salve配置 `broker-b-s.properties`:

```properties
namesrvAddr=ip1:9876;ip2:9876
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=1
deleteWhen=04
fileReservedTime=48
# SLAVE
brokerRole=SLAVE 
flushDiskType=ASYNC_FLUSH
listenPort=11011
storePathRootDir=/Users/mac/Documents/AllMyDocuments/RocketStore1
```

> 注意存储目录不同，还有关 RocketMQ 相关的配置参数请参考[相关配置参数](http://rocketmq.apache.org/docs/rmq-deployment/)
>
> 物理机2与之类似,唯一不同是，SLAVE 分别在对方那里(疑惑的地方🤔️)

最后分别使用启动命令启动这 4个 broker

```shell
$ nohup sh  bin/mqbroker -c config_file & # config_file 是配置文件的路径
$ nohup sh bin/mqbroker -c conf/2m-2s-sync/broker-a.properties &# 示例1
$ ...
```

## 3.发送/接受消息示例

下面同步方式的 Java 代码示例：

```java
// 发送消息
public class SyncProducer {
    public static void main(String[] args) throws Exception {
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new
                DefaultMQProducer("please_rename_unique_group_name");
        // Specify name server addresses.
        producer.setNamesrvAddr("localhost:9876");
        //Launch the instance.
        producer.start();
        for (int i = 0; i < 100; i++) {
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " +
                            i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            //Call send message to deliver message to one of brokers.
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
}


//接受消息
public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {

        // Instantiate with specified consumer group name.
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name");

        // Specify name server addresses.
        consumer.setNamesrvAddr("localhost:9876");

        // Subscribe one more more topics to consume.
        consumer.subscribe("TopicTest", "*");
        // Register callback to execute on arrival of messages fetched from brokers.
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                            ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        //Launch the consumer instance.
        consumer.start();

        System.out.printf("Consumer Started.%n");
    }
}
```

> 必须配置：
>
> - GroupName
> - NameServer
> - Port
> - Topic
>
> 最后进入相关逻辑：目前 GroupName 和 Topic 尚不清楚

## 4. 图形化管理集群

1. [下载rocketmq-externals的 rocketmq-console](https://github.com/apache/rocketmq-externals/tree/master/rocketmq-console)
2. 配置信息
3. 启动
4. 网页访问 `localhost:8080`

项目`application.properties`文件：

```properties
rocketmq.config.namesrvAddr=ip1:9876;ip2:9876
```

> ⚠️注意：
>
> 1. 需要在`conf/2m-2s-sync`目录的配置文件添加 `brokerIP1=ip`否则连接失败
> 2. 还需要注意服务器打开端口

# 相关链接

- [RocketMQ官方链接](http://rocketmq.apache.org/)



# 相关技巧

Mac 获取公网 IP：

```shell
curl http://members.3322.org/dyndns/getip
```

