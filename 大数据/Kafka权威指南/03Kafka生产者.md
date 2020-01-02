# Kafka 生产者-向 Kafka 写入数据

不管是把 Kafka 作为**消息队列**、**消息总线**还是**数据存储平台**来使用，总是需要有一个：

- 往 Kafka 写入数据的生产者和一个可以从 Kafka 读取数据的消费者
- 或者一个兼具两种角色的应用程序

> 开发者可以使用 Kafka 内置的客户端 API 开发 Kafka 应用程序

本章内容：

- 演示创建 KafkaProducer 和 ProducerRecords 对象
- 如何将记录发送给 Kafka ，以及如何处理从 Kafka 返回的错误，然后介绍用于控制生产者行为的重要配置选项
- 使用不同的**分区方法和序列化器**，如何自定义序列化器和分区器

> **第三方客户端：**除了内置的客户端外，Kafka 还提供了二进制连接协议，也就是说，我们直接向 Kafka 网络端口发送适当的字节序列，就可以实现从 Kafka 读取消息或往 Kafka 写入消息。还有很多用其他语言实现的 Kafka 客户端，比如 C++、Python、Go 语言等

# 生产者概述

一个应用程序在很多情况下需要往 Kafka 写入消息：

- 记录用户的活动(用于审计和分析)
- 记录度量指标
- 保存日志消息
- 记录智能家电的信息
- 与其他应用程序进行异步通信
- 缓冲将写入到数据库的数据

这些信息会有许多的使用场景：

- 是否允许丢失消息

- 偶尔出现重复消息是否可以接受

- 是否可以接受延迟

	![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1gadz5ward3j30sy0lstb2.jpg)

ProducerRecord 对象需要包含目标主题和要发送的内容。在发送 ProducerRecord 对象时，生产者要先把键和值对象序列化成字节数组，这样它们才能够在网络上传输。接下来，数据被传给分区器。如果没有指定分区，那么分区器会根据 ProducerRecord 对象的**键来选择一个分区**。选好分区以后，生产者就知道该往**哪个主题和分区发送到相同的主题**和分区上。有一个独立的线程负责把这些记录批次发送到相应的 broker 上

服务器在收到这些消息时会**返回一个响应**。如果消息成功写入 Kafka，就返回一个 RecordMetaData 对象，它包含了主题和分区信息，以及记录在分区里的偏移量。如果写入失败，则会返回一个错误。生产者在收到错误之后会尝试重新发送消息，几次之后如果还是失败，就返回错误消息

# 创建 Kafka 生产者

要往 Kafka 写入信息，首先要创建一个**生产者对象**，并设置一些属性。Kafka 生产者有 3 个必选的属性

**bootstrap.servers**

该属性指定 broker 的地址清单，地址的格式为 host:port。清单里不需要包含所有的 borker 地址，生产者会从给定的 broker 里查找到其他 broker 的信息。不过创建**至少要提供两个 broker 的信息**，一旦其中一个宕机，生产者任然能够连接到集群上

**key.serializer**

broker 希望接收到的消息的键和值都是**字节数组**。生产者接口允许使用**参数化类型**,因此可以把 Java 对象作为键和值发送给 broker。这样的代码具有良好的可读性，不过生产者需要知道如何把这些 Java 对象转换成字节数组。**key.serializer 必须设置为一个实现了 `org.apache.kafka.common.serialization.Serializer`🤔 接口的类**

**value.serializer**

与 key.serializer 一样，value.serializer 指定的类会将值序列化。如果键和值都是字符串，可以使用与 key.serializer 一样的序列化器。如果**键是字符串**，那么需要**使用不同的序列化器**。



如果和创建一个新的生产者，这里只指定了必要的属性，其他使用默认设置

```java
private Properties kafkaProps = new Properties();
kaflaProps.put("bootstrap.servers","broker:9092,broker2:9092");

kafkaprops.put("key.serializer","org.Apache.Kafka.Common.serialization.Stringserializer");
kafkaprops.put("value.serializer","org.Apache.Kafka.Common.serialization.Stringserializer");

producer =new KafkaProducer<String,String>(kafkaprops);
```

- 新建一个 Properties 对象
- 因为我们打算把键和值定义成**字符串类型**，所以使用内置的 StringSerializer
- 在这里我们创建了一个**新的生产者对象**，并为键和值设置了恰当的类型，然后把 Properties 对象传给它

实例化生产者对象后，接下来就可以开始发送消息了。发送消息主要有 3 种方式

- 发送并忘记(fire-and-forget)

- 同步发送

	> 我们使用 send( ) 方法发送消息，它会返回一个 Future 对象,调用 get() 方法进行等待,就可以知道是否发送成功

- 异步发送

	> 我们调用 send() 方法,并指定一个回调函数,服务器在返回响应时调用该函数

> ⚠️注意：本章的所有例子都使用**单线程**,但其实生产者是可以使用多线程来发送消息的。刚开始的时候可以使用单个消费者和单个线程。如果需要更高**的吞吐量**，可以在生产者数量不变的前提下**增加线程数量**。

# 发送消息到 Kafka

最简单的消息发送方式:

```java
ProducerRecord<String,String> record = new ProducerRecord<>("CustomerCountry","Precision Products","France");
try{
    producer.send(record);
}catch(Exception e){
    e.printStackTrace();
}

```

- 生产者的 send() 方法将 ProducerRecord 对象作为参数，所以我们要先创建一个 ProducerRecord 对象。ProducerRecord 有多个构造函数，稍后我们会详细讨论。这个构造方法使用(主题、key、value)
- 发送 ProducerRecord 对象。从生产者的架构图里可以看到，**消息先是被放进缓冲区**，然后使用单独的**线程发送到服务器**端。send() 方法会返回一个包含 RecordMetadata 的 Future 对象，如果不关心返回结果，就是用这种方式

## 同步发送消息

最简单的同步发送消息方式：

```Java
ProducerRecord<String,String> record = new ProducerRecord<>("CustomerCountry","Precision Products","France");
try{
    producer.send(record).get();
}catch(Exception e){
    e.printStackTrace();
}
```

- 在这里，producer.send() 方法先返回一个 Future 对象，然后调用 Future 对象的 get() 方法**等待 Kafka 响应**。如果服务器返回错误，get() 方法会抛出异常。如果没有发生错误，我们会得到一个 RecordMetadata 对象，可以用**它获取消息的偏移量**



kafkaProducer 一般会发生两类错误。

- 可重试异常
- 直接抛出异常

## 异步发送消息

🌰例子：如果是同步发送消息，发送一个消息的来回是 10ms。如果在发送完每个消息后等待回应，100个消息就是 1s。如果只发送不等待响应，那么发送 100 所需要的时间会少很多，我们并不需要等待响应,**尽管 kafka 会把目标主题、分区信息和消息的偏移量发送回来，但对于发送端的应用程序来说不是必需的。**

当遇到消息发送失败时，我们需要抛出异常、记录错误日志，或者把消息写入“错误消息”文件以便日后分析



```java
private class DemoProducerCallback implements Callback{
    @Override
    public void onCompletion(RecordMetadata recordMetadata,Exception e){
        if(e != null){
            e.printStackTrace();
        }
    }
}

ProducerRecord<String,String> record = new ProducerRecord<>("CustomerCountry","Biomedical Materials","USA");
producer.send(record,new DemoProducerCallback());
```

- 为了使用回调，需要一个实现了 `org.apache.kafka.clients.producer.Callback`接口的类，这个接口只有一个 onCompletion 方法
- 如果 Kafka 返回一个错误，onCompletion 方法会抛出一个非空(non null) 异常
- 记录与之前的一样
- 在发送消息时传进去一个回调对象

