# 目录

# Redis 简介
Redis 是一个速度非常快的**非关系数据库**,它可以存储键(key)与 5 种不同类型的值(value)之间的映射,可以将存储在内存的键值对数据持久化到硬盘,可以使用复制特性来扩展读性能,还可以使用客户端分片来扩展写性能

> 通过对数据进行分片,用户可以将数据存储到多台机器里面,也可以从多台机器里面读取数据,这种方法在解决某些问题时可以获得线性级别的性能提升

**Redis与其他数据库和软件的对比**

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g398iabxpnj30xn0kj166.jpg)

**附加特性**

Redis 拥有两种不同形式的持久化方法,它们都可以用小而紧凑的格式将存储在内存中的数据写入硬盘:
1. 时间点转储,在指定时间段内有指定数量的写操作执行
2. 所有修改了数据库的命令都写入一个只追加文件里面,用户可以根据数据的重要程度,将只追加写入设置为从不同步(sync)、每秒同步一次或者和每写入一个命令就同步一次


尽管 Redis 性能很好,但受限与 Redis 的内存存储设计,使用一台 Redis 服务器可能没有办法处理所有请求。为了 Redis 提供故障转移支持, Redis 实现**主从复制特性**:执行复制的从服务器会连接上主服务器,接收主服务器发送的整个初始副本(copy);之后主服务器执行写命令,都会被发送给所有连接着的从服务器去执行,从而实时更新从服务器的数据集

# Redis 数据结构简介

5 种不同数据结构类型:
- STRING
- LIST
- SET
- HASH
- ZSET (有序集合)

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g39914bpr5j30wz0dhk12.jpg)

## Redis 中的字符串

字符串拥有一些和其他键值存储相似的命令,比如 GET(获取值)、SET(设置值)和 DEL (删除值)
![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g399b5of60j30wv04xn0b.jpg)
> redis-cli 这个客户端来介绍 Redis 命令

```
<!-- 进入 redis-cli -->
redis-cli

set hello world   //将键 hello 的值设置为 world
get hello          //获取键 hello 的值
del hello           //删除这个键值对
nil         //键值删除得到 nil,Python 将这个 nil 转换成 None

```

## Redis 中的列表
Redis 对链表(Linked-list)结构的支持使得它在键值存储的世界独树一帜。一个列表结构可以**有序存储多个字符串**,Redis 列表可执行的操作和很多编程语言里面的列表操作非常相似:LPUSH 命令和 RPUSH 命令分别用于将元素**推入列表的左端和右端**;LPOP 命令和 RPOP 命令分别用于从列表的**左端和右端弹出元素**;LINDEX 命令用于获取列表在给定位置上的元素;LRANGE 命令用于获取列表在给定范围上的所有元素。

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3abcvwfw6j30wv05x0vr.jpg)

```
<!-- RPUSH LRANGGE LINDEX LPOP 使用示例 -->
rpush list-key item  
rpush list-key item2
rpush list-key item   // 前三个向列表插入元素
lrange list-key 0 -1  // 使用 0 索引范围,-1 为范围的结束索引,可以取出列表所有的元素
lindex list-key 1     // 使用 LINDEX 可以从元素取出单个元素
lpop list-key         // 从列表弹出一个元素,被弹出的元素将不再存在于列表

```
Redis 列表还拥有从列表里面移除元素的命令,将元素插入列表中间的命令、将列表修剪至指定长度等等

## Redis 中的集合
Redis 的集合和列表都可以存储多个字符串,它们之间的不同在于,列表可以存储多个**相同的字符串**,而集合则通过使用**散列表保证自己存储的每个字符串都是各不相同的(这些散列表只有键,但没有与键相关联的值)。本书表示集合的方法和表示列表的方法基本相同。

因为 Redis 的集合使用**无序方式存储元素**,所以用户不能像使用列表那样,将元素推入集合的某一端,或者从集合的某一端弹出元素。而使用 SADD 命令将元素添加到集合,或者使用 SREM 命令从集合里面移除元素。还可以通过 SISMEMBER 命令快速地检查一个元素是否已经**存在于集合中**,或者使用 SMEMBERS 命令获取集合包含的所有元素(如果集合包含非常多元素非常多,那么 SMEMBERS 会很慢)

![](http://ww1.sinaimg.cn/mw690/006rAlqhgy1g3acztwgw5j30wp061ad4.jpg)

```
<!-- SADD SMEMBERS Sismenber 和 SREM 的使用示例 -->
sadd set-key item
sadd set-key item2
sadd set-key item3    
sadd set-key item    // 尝试将一个元素添加到集合的时候,命令返回 1 表示这个元素被成功地添加到了集合里面
smembers set-key    // 获取集合包含的所有元素将得到一个由元素组成的序列,Python 客户端会将这个序列转换成 Python 集合

sismember set-key item4    //检查一个元素是否存在于集合中,Python 客户端会返回一个布尔值来表示检查结果

srem set-key item2     //在使用命令移除集合中的元素时
```

## Redis 中的散列

Redis 的散列可以**存储多个键值对之间的映射**。和字符串一样,散列存储的值既可以是**字符串**又可以是**数字值**,并且用户同样可以对散列存储的数字值执行**自增操作或者自减操作**。散列在很多方面就像是一个微缩版的 Redis,不少字符串命令都有相应的散列版本。下面对散列执行插入元素、获取元素和移除元素等操作

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3adj36n3aj30wx062act.jpg)

```
<!-- 散列命令 -->
hset hash-key sub-key1 value1 // 存储散列表 hash-key ,键值对为 sub-key1 value1
hset hash-key sub-key2 value2
hset hash-key sub-key1 value1 //无法插入因为已经存在
hgetall hash-key   // 获取散列包含的所有键值对时,python 客户端会把整个散列表转换成一个 Python 字典

hdel hash-key sub-key1      // 删除整个键值对

```

## Redis 中的有序集合
有序序列和散列一样,都用于存储键值对;有序集合的键被称为成员(member),每个成员都是各不相同；而有序集合的值则被称为分值(score),**分值必须为浮点数**有序集合是 Redis 里面唯一一个既可以根据成员访问元素,又可以根据分值以及分值的排列顺序访问元素结构

![](http://ww1.sinaimg.cn/large/006rAlqhly1g3g82w37uwj30g4031js5.jpg)

```
<!-- 有序集合命令 -->
zadd zset-key 728 member1
zadd zset-key 982 member0 //尝试向有序序列添加元素
zrange zset-key  0 -1 withscores // 获取有序集合包含的所有元素,多个元素按照分值大小进行排序,并且 python 客户端会将元素的分值转换成浮点数
zrangebyscore zset-key 0 800 withscores //用户根据分值范围获取一部分元素
zrem zset-key member1 //移除有序集合
```

# 你好 Redis
