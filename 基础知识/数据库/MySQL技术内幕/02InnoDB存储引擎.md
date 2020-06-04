# InnoDB存储引擎

> InnoDB 是事务安全的 OLTP 应用的存储引擎，本章介绍 InnoDB 的体系结构和其他存储引擎的区别

## 1.InnoDB存储引擎概述

InnoDB 特点

- 第一个完整支持 ACID 事务的 MySQL 存储引擎
- 支持行锁设计、支持 MVCC、支持外键、提供一致性非锁定读
- 被设计最有效地利用以及使用内存和 CPU

> InnoDB 目前是一个高性能、高可用、高可扩展的存储引擎

## 2.InnoDB存储引擎的版本

| 版本                      | 功能                                            |
| ------------------------- | ----------------------------------------------- |
| 老版本的 InnoDB(静态编译) | 支持 ACID、行锁设计、MVCC                       |
| InnoDB 1.0.x              | 继承上述版本，增加了 compress 和 dynamic 页格式 |
| InnoDB 1.1.x              | 继承上述版本，增加了 Linux AIO、多回滚段        |
| InnoDB 1.2.x              | 继承上述版本，增加了全文索引支持、在线索引添加  |

## 3.InnoDB体系架构

体系结构图:

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1gblq76ln4gj30zo0lgdik.jpg)

### 后台线程：

> - 负责刷新内存池中的数据，保证内存缓存是最新的数据。
> - 此外将修改的数据文件刷新到磁盘文件，同时保证在数据库发生异常的情况下 InnoDB 能恢复到正常运行状态

| 后台线程              | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| 1.Master Thread       | 主要负责将**缓冲池中的数据**异步刷新到磁盘**，保证数据一致性，包括脏页的刷新、合并插入缓存、undo 页的回收等 |
| 2.IO Thread           | 使用 AIO （异步 IO），这样可以极大提高数据库的性能。<br />read thread 和 write thread 分别增大到了 4 个，使用 `innodb_read_io_threads`和`innodb_write_io_threads`参数进行设置 |
| 3.Purge Thread        | purge 操作独立到单独的线程中进行,从而提高 CPU 的使用率以及存储引擎的性能。<br />通过在 MySQL 配置中添加 `innodb_pruge_threads=1` |
| 4.Page Cleaner Thread | 将之前版本中脏页的刷新操作都放入到单独的线程中来完成。       |

该过程用到的命令

```mysql
# 显示 innodb_version 版本
mysql> show variables like 'innodb_version'
# 显示 io 的数量
mysql> show variables like 'innodb_%io_threads';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| innodb_read_io_threads  | 4     |
| innodb_write_io_threads | 4     |
+-------------------------+-------+
2 rows in set (0.00 sec)
# 观察 InnoDB 中的 IO Thread
mysql> show engine innodb status\G;
# 显示 purge 线程的数量
mysql> show variables like 'innodb_purge_threads';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| innodb_purge_threads | 4     |
+----------------------+-------+
1 row in set (0.00 sec)
```

### 内存：

| 内存                                | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| 缓冲池                              | 基于**磁盘存储**、按照**页的方式**进行管理。<br />通过内存速度来弥补磁盘速度较慢对数据库的影响<br />页修改需要刷新回磁盘需要通过**Checkpoint**的机制刷新回磁盘<br />通过`innodb_buffer_pool_size`来设置缓冲池大小的配置<br />允许有多个缓冲池实例，增加数据库的并发能力<br />mysql 的 information_schema 数据库下的 `innodb_buffer_pool_stats`表显示缓冲的状态 |
| **LRU 列表、FREE 列表、Flush 列表** | 数据库缓冲池是通过 LRU(最近最少使用算法) 来进行**管理**的<br />缓冲池**页的大小**默认 16KB，在 InooDB 存储引擎对传统的 LRU 列表中加入了 midpoint 位置作为插入点，通过 `innodb_old_blocks_pct`查询，这 midpoint 之前为 new 列表，之后为 old 列表<br />还通过`innodb_old_blocks_time`参数管理，用于读取到 mid 位置后需要**等待多久才会被加入到 LRU 列表的热端**。防止热点数据被普通数据刷出<br />如果热点数据不止 63%，那么执行 SQL 语句前减少热点页减少被刷出概率<br />Flush 列表中的页即位**脏页**，也就是缓冲池修改的页与磁盘不一致 |
| 重做日志缓冲                        | 每一秒钟会将重做日志缓冲刷新到日志文件，通过`innodb_log_buffer_size`控制，默认 8MB。下列三种情况会将内容刷新**外部磁盘**日志文件中<br />1. Master Thread 每一秒将缓冲日志刷新到文件中<br />2. 每个事务提交时会将重做日志缓冲刷新<br />3. 重做缓冲池剩余空间小于 1/2 时 |
| 额外的内存池                        | 暂不明白                                                     |

用到的命令：

```mysql
# 查询缓冲池大小
# 此处缓冲池数据页类型：索引页、数据页、undo 页、插入缓冲、自适应哈希索引、InnoDB存储的锁信息(lock info)、数据字典等
mysql> show variables like 'innodb_buffer_pool_size';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.00 sec)
# 缓冲池实例
mysql> show variables like 'innodb_buffer_pool_instances';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| innodb_buffer_pool_instances | 1     |
+------------------------------+-------+
1 row in set (0.01 sec)
# 通过 information_schema 自带数据库的具体表查看缓冲池状态
mysql> use information_schema
mysql> select POOL_ID,POOL_SIZE,FREE_BUFFERS,DATABASE_PAGES
-> from INNODB_BUFFER_POOL_stats;

# 显示 LRU old 列表，可以
mysql> show variables like 'innodb_old_blocks_pct';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_old_blocks_pct | 37    |
+-----------------------+-------+
1 row in set (0.00 sec)
# LRU 等待多节插入热端？
mysql> show variables like 'innodb_old_blocks_time';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_old_blocks_time | 1000  |
+------------------------+-------+
1 row in set (0.00 sec)
# 重做日志缓冲参数 
mysql> show variables like 'innodb_log_buffer_size';
+------------------------+----------+
| Variable_name          | Value    |
+------------------------+----------+
| innodb_log_buffer_size | 16777216 |
+------------------------+----------+
1 row in set (0.00 sec)

```

> LRU 中的 midpoint 是因为有些操作查询的并不是活跃的数据，如果插入到收不，就可能把需要的热点数据从列表中移除，InnoDB 还需要访问磁盘

**数据库启动时，LRU 列表是空的。当需要从缓冲池中分页时，首先从 Free 列表中查找是否有可用的空闲页，若有则将该页从 Free 列表中删除，放入到 LRU 列表中。否则淘汰 LRU 尾部页。**

- 当页从 LRU 列表的 old 部分加入 new 部分时，称此时发生的操作 `page made young`
- 因为`innodb_old_blocks_time`而没有转移的称为 `page not made young`

## 4.Checkpoint 技术