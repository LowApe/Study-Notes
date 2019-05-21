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
/* 进入 redis-cli */
redis-cli

set hello world   //将键 hello 的值设置为 world
get hello          //获取键 hello 的值
del hello           //删除这个键值对
nil         //键值删除得到 nil,Python 将这个 nil 转换成 None

```

## Redis 中的列表
## Redis 中的集合
## Redis 中的散列
## Redis 中的有序集合
