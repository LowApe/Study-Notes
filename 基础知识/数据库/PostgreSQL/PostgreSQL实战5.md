# PostgreSQL 体系结构

> 通常将物理文件称为数据库，将这些物理文件、管理这些物理文件的进程、进程管理的内存称为这个数据库的实例。内部功能实现上，可以分为：
>
> - 系统控制器：负责接收外部连接请求
> - 查询分析器：对连接请求查询进行分析并生成优化后的查询解析树
> - 事务系统：
> - 恢复系统：
> - 文件系统：获取结果集或通过事务系统对数据做处理并由文件系统持久化数据

# 逻辑和物理存储结构

> 数据库集群，是指单个 PostgreSQL 服务器实例管理的数据库集合，组成数据库集群这些数据库**使用相同的全局配置文件和监听端口、共用进程和内存结构**

## 逻辑存储结构

![image.png](https://i.loli.net/2020/09/30/vr8d3xtTP4cinSV.jpg)

## 物理存储结构

> 数据库的文件默认保存在 initdb 时创建的数据目录中。在**数据目录中有很多类型、功能不同的目录和文件，除了数据文件之外，还有参数文件、控制文件、数据库运行日志及预写日志等**

### 1.数据目录结构

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8grh2yd8j30cq0ym41y.jpg)

|      目录      |                    用途                    |
| :------------: | :----------------------------------------: |
|      base      |         包含每个数据库对应的子目录         |
|     gloabl     | 包含集群范围的表的子目录，比如 pg_database |
|  pg_commit_ts  |         事务提交时间戳数据的子目录         |
|    pg_xact     |          事务提交状态数据的子目录          |
|  pg_dynshmem   |   被动态分享内存子系统所使用文件的子目录   |
| **pg_logical** |       **用于逻辑复制的状态的子目录**       |
|     pg_wal     |                保存预写日志                |

|         文件         |                     用途                     |
| :------------------: | :------------------------------------------: |
|      PG_VERSION      |           PostgreSQL 主版本号文件            |
|     pg_hba.conf      |              客户端认证控制文件              |
|   postgresql.conf    |                   参考文件                   |
| postgresql.auto.conf | 参考文件，只保存 alter system 命令修改的参数 |
|   postmaster.opts    |   记录服务器最后一次启动时使用的命令行参数   |

### 2.数据文件布局

> 数据目录中的 **base 子目录是我们数据文件默认保存的位置，是数据库初始化后的默认表空间**。需要了解的基础的数据库对象

#### **OID**：

数据库对象都由各自的对象标识符(OID)进行内部管理，它们是无符号的 4 字节整数。数据库的 OID 存储在 pg_database 系统表中.数据库中的表、索引、序列等对象的 OID 存储在 pg_class 系统表中

#### **表空间**：

数据库中创建的对象都保存在表空间.在创建数据库对象时，可以指定数据库对象的表空间，如果不指定则使用默认表空间，默认为 pg_default 表空间

- pg_global 表空间的物理文件位置在数据目录的 global 目录中，它用来保存系统表

- pg_default 表空间物理文件位置在数据目录中的 base 目录,是 temp late0 和 template 1 数据库的默认表空间，我们知道创建数据库时，默认从 template1 数据库进行克隆，因此除非特别指定了新建数据库的表空间，默认使用 template1 的表空间，也就是 pg_default

```sql
-- 自定义表空间
-- 先创建表空间路径
mkdir -p /pgdata/12/mytblspc

-- 连接数据库并创建表空间
psql -p 5432 mydb

create tablespcae myspc location '/pgdata/12/mytblspc';
CREATE TABLESPACE
mydb=# \db
                                              List of tablespaces
    Name    | Owner |                                         Location
------------+-------+------------------------------------------------------------------------------------------
 pg_default | mac   |
 pg_global  | mac   |
 myspc  | mac   | /pgdata/12/mytblspc
(3 rows)

-- 当创建新的数据库或表时，便可以指定刚才创建的表空间：
create table t(id serial primary key,iva int )
tablespace myspc;
CREATE TABLE
```

> 由于表空间定义了存储的位置，在创建数据库对象时，会在当前的表空间目录**创建一个以数据库 OID 命名的目录**

#### **数据文件命名**

> PostgreSQL 每个表和索引都用一个文件存储，新创建的表文件以表的 OID 命名，对于大小超过 1GB 的表数据文件，会自定**将其切分为多个文件来存储,切分的文件以 `OID.<顺序号>`来命名

```sql
-- city 表的 oid 和 relfilenode 相同
mydb=# select oid,relfilenode from pg_class where relname = 'city';
  oid  | relfilenode
-------+-------------
 16571 |       16571
(1 row)

-- truncate 清空 city 表的所有数据
truncate city;
TRUNCATE TABLE
-- 强制指定预定义日志检查点
checkpoint;
CHECKPOINT
mydb=# select oid,relfilenode from pg_class where relname = 'city';
  oid  | relfilenode
-------+-------------
 16571 |       16588
(1 row)

-- 插入数据，查看文件名称
mydb=# insert into city select ('a','b') from generate_series(1,20000000);
INSERT 0 20000000
mydb=# insert into city select ('a','b') from generate_series(1,10000000);
INSERT 0 10000000

```

清空前：

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8ivroav0j30yu01e3yj.jpg)

清空后：

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8ix97gjlj30x201saa8.jpg)

写入数据后:

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8j2n387sj30yo03674v.jpg)

> city 表的大小超过 1GB，city 表的 relfilenode 为 16588，超过1GB之后，名称为 16588.1.
>
> - _fsm 文件是空闲空间映射表文件，包含可用的空间
> - _vm 文件是可见性映射表文件。包含已知对所有活动事务可见的元组

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8j9am7stj30x00jqthb.jpg)

> 上图pg_tblspc 是默认表空间，如果设置自定义的表空间，去对应自定路径查看



#### **表文件内部结构**

在 PostgreSQL 中，

- 将保存在磁盘中的块称为 Page。每个 Page 默认大小为 8kB，指定 BLCKSZ大小决定 Page 的大小，每个Page包含若干 Tuple
- 而将内存中的块称为 Buffer
- 表和索引称为 Relation
- 行称为 Tuple

 

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8msmat41j30xe0cyad9.jpg)

PageHeader 描述了一个数据页的页头信息，包含页的一些元信息

- pd_lsn：确定和记录最后更改此页的 xlog 记录的 LSN，把数据页和 WAL 日志关联，用于恢复数据时校验日志文件和数据文件的一致性
- pg_flags:标识页面的数据存储情况
- pd_special:指向索引相关数据的开始位置，该项在数据文件中为空，主要是针对不同索引
- pd_lower：指向空闲空间的起始位置
- pd_upper: 指向空闲位置的结束位置
- pd_pagesize_version: 不同的 PostgreSQL 版本的页的格式可能会不同
- pd_linp[1]: 行指针数组，即的Item1，Item2，这些地址指向 Tuple 的存储位置

> 当从数据库中检索数据时有两种典型的访问方法，顺序扫描和B树索引扫描。
>
> - 顺序扫描通过扫描每个页面中的所有行指针顺序读取所有页面中的所有元组
> - B 树索引扫描时，索引文件包含索引元组，每个元组由索引建和指向目标堆元组的 TID 组成。如果找到了正在查找的健的索引元组，使用获取到的 TID 读取所需的堆元组

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8ozwvezej30x80u8k0i.jpg)



![image-20200930150851710](/Users/mac/Library/Application Support/typora-user-images/image-20200930150851710.png)

# 进程结构

> PostgreSQL 是一用户一进程的客户端/服务端.数据库启动时启动若干个进程：
>
> - postmaster(守护进程)
> - postgres（服务进程）
> - syslogger
> - checkpointer
> - bgwriter
> - walwriter

## 守护进程与服务进程

首先从 postmaster（守护进程）。postmaster 进程的主要职责有：

- 数据库的启停
- 监听客户端连接
- 为每个客户端连接 fork 单独的 postgres 服务进程
- 当服务进程出错时进行修复
- 管理数据文件
- 管理与数据库运行相关的辅助进程

> 当客户端**接口库**向数据库发起连接请求，守护进程 postmaster 会 fork **单独的服务进程 postgres 为客户端提供服务**，此后将由 postgres 进程为客户端执行各种命令，**不需要 postmaster 中转，直接与服务进程 postgres 通信**

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8pc859bgj30wi08cmz5.jpg)

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8pkljmn0j30ye0tsafk.jpg)



## 辅助进程

- background writer: 称 bgwriter 进程,bgwriter 进程很多时候都是在休眠状态，**每次唤醒后它会搜索共享缓冲池找到被修改的页，并将它们从共享缓冲池刷出**
- autovacuum launcher: 自动清理回收垃圾进程
- WAL writer：定期将 WAL 缓冲区上的 WAL 数据写入磁盘
- statistics collector：统计信息收集进程
- logging collector：日志进程，将消息或消息或错误信息写入日志
- archiver: WAL 归档进程
- checkpointer: 检查点进程

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8pn5ug1ej30wi0gidkc.jpg)

# 内存结构

- 本地内存
- 共享内存
- 辅助进程分配的内存

## 本地内存

> 当后端服务进程被 fork 时，每个后端进程为查询分配一个本地内存区域。本地内存由三部分组成
>
> - work_mem：当使用 order by 或 distinct 操作对元组进行排序时会使用这部分内存
> - maintenance_work_mem：维护操作，例如 vacuum、redindex、create index 等操作使用这部分内存
> - temp_buffers：临时表相关操作使用这部分内存

## 共享内存

> 共享内存在 PostgreSQL 服务器启动时分配，由所有后端进程共同使用。共享内存由三部分组成：
>
> - shared buffer pool:PostgreSQL 将表和索引中的页面从持久存储装载到这里，并直接操作它们
> - WAL buffer： WAL 文件持久化之前到缓冲区
> - CommitLog buffer：PostgreSQL 在 Commit Log 中保存事务的状态，并将这些状态保留在共享内存缓冲区中，在整个事务处理过程中使用。

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gj8q04jmwhj30y80s0dnl.jpg)

# 小结

- PostgreSQL 数据库的文件存储，构成数据库的参数文件、数据文件布局，以及表空间、数据库、数据库对象的逻辑存储结构
- PostgreSQL 的守护进程、服务进程、辅助进程