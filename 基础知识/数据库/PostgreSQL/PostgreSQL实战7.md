# 事务与并发控制

> 为了控制并发事务之间的相互影响，解决并发可能带来的资源征用及数据不一致问题，数据库的**并发控制系统引入了基于锁的并发控制机制(Lock-Based Concurrency Control)和基于多版本的并发控制机制 MVCC(Multi-Version Concurrencu Control)**

# 1 事务和并发控制的概念

事务的四个重要的特性

- 原子性 (要么全部执行，要么全部不执行)
- 一致性
- 隔离性（确保事务与事务并发执行正常）
- 持久性

## 并发引发的现象

| 现象       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 脏读       | 第一个事务读取了第二个事务已经修改但还未提交的数据<br />如果第二个事务不提交并执行了 ROLLBACK 后，<br />第一个事务读取数据是不正确的，这种现象称作脏读 |
| 不可重复度 | 当一个事务第一次读取数据之后，被读取的数据被另一个已提交的事务进行了修改，事务再次读取这些数据时发现两次查询结果不一致。**同一条记录的值不同了** |
| 幻读       | 一个事务的两次查询的**结果集记录数不一致**。                 |
| 序列化异常 | 成功提交的一组事务的执行结果与这些事务按照执行方式的执行结果不一致 |

> 注意⚠️：
>
> **PostgreSQL 数据库中无论如何都无法产生脏读**：由于内部将 read uncommited 设计为和 read commited 一样，:sos:：这块有关隔离级别



## ANSI SQL 标准的事务隔离级别

> 为了避免事务与事务之间并发执行引发的副作用，最简单的方法是**串行化地逐个执行事务，但是会降低系统吞吐量等** ANSI SQL 标准定义了四类隔离级别。**通过这些事务隔离界别规定一个事务必须与其他事务所进行的资源或数据更改相隔离的程度**

| 隔离级别                    | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| Read Uncommitted (读未提交) |                                                              |
| Read Committed(读已提交)    | PostgreSQL 的默认隔离级别，它满足了一个事务只能看见已经提交了 |
| Repeatable Read(可重复度)   | 确保同一个事务的多个实例在并发读取数据时，会看到同样的数据行 |
| Serializable(可序列化)      | 最高的隔离级别，**通过强制事务排序，使之不可能相互冲突，从而解决幻读问题** 它是在每个读的数据行上加上共享锁 |



PostgreSQL 隔离级别与读现象的关系：

| 隔离级别                    | 脏读   | 不可重复度 | 幻读   | 序列化异常 |
| --------------------------- | ------ | ---------- | ------ | ---------- |
| Read Uncommitted (读未提交) | 不可能 | 可能       | 可能   | 可能       |
| Read Committed(读已提交)    | 不可能 | 可能       | 可能   | 肯能       |
| Repeatable Read(可重复度)   | 不可能 | 不可能     | 不可能 | 可能       |
| Serializable(可序列化)      | 不可能 | 不可能     | 不可能 | 不可能     |

> 对多数应用程序，优先考虑 Read Commited 隔离级别。它能够避免脏读，而且具有较好的并发性能。尽管会导致**不可重复读、幻读和和丢失更行这些并发问题，在这类问题的个别场合，可以由应用程序采用悲观锁或乐观锁来控制**



# 2 PostgreSQL 的事务隔离级别

**试验 隔离级别 可重复度 PostgreSQL 没有出现幻读**

| T1                                                           | T2                                                    |
| ------------------------------------------------------------ | ----------------------------------------------------- |
| mydb=# begin transaction isolation level repeatable read;<br />BEGIN<br />mydb=# select * from tbl_mvcc where id > 3 and id < 10;<br />id ival<br />4  4<br />5  5 |                                                       |
|                                                              | BEGIN;<br />inset into tbl_mvcc values(6,6)<br />end; |
| select * from tbl_mvcc where id>3 and id < 10;<br />id ival<br />4 4<br />5 5<br />end; |                                                       |

## 1 查看和设置数据库的事务隔离级别

> PostgreSQL 默认的事务隔离级别是 Read Committed 查看全局事务隔离级别的代码如下所示：

```sql

mydb=# select name,setting from pg_settings where name = 'default_transaction_isolation';
             name              |    setting
-------------------------------+----------------
 default_transaction_isolation | read committed
(1 row)
```

## 2 修改全局的事务隔离级别

**方法1：**通过修改`postgresql.conf`文件中的 `default_transaction_isolation` 参数修改全局事务隔离级别，修改之后  reload 示例使之生效



**方法2：**通过 `alter system` 命令修改全局事务隔离级别：

```sql
mydb=# alter system set default_transaction_isolation to 'repeatable read';
alter system
-- 使配置生效
mydb=# select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)
-- 查看当前事务的隔离级别
mydb=# select current_setting('transaction_isolation');
 current_setting
-----------------
 read committed
(1 row)

```



## 3 查看当前回话的事务隔离级别

```sql
-- 查看当前事务的隔离级别
mydb=# select current_setting('transaction_isolation');
 current_setting
-----------------
 read committed
(1 row)
```



## 4 设置当前会话的事务隔离级别

```sql
mydb=# set session characteristics as transaction isolation level read uncommitted;
SET
mydb=# show transaction_isolation;
 transaction_isolation
-----------------------
 read uncommitted
(1 row)

```

## 5 设置当前事务的事务隔离级别

> 在启动事务的同时设置事务隔离级别，如下所示：

```sql
mydb=# start transaction isolation level read uncommitted;
START TRANSACTION

# 操作...

mydb=# end;
COMMIT
mydb=#
```

或者

```sql
mydb=# begin isolation level read uncommitted read write;
BEGIN
mydb=# end
mydb-# ;
COMMIT
mydb=# rollback
mydb-# ;
WARNING:  there is no transaction in progress
ROLLBACK	
```



# 3 PostgreSQL 的并发控制

> 在多用户环境中，允许多人同时访问和修改数据，为了保持事务的隔离性，系统必须对并发事务之间的相互作用加以控制。
>
> 数据库管理系统中并发控制的任务便是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性、数据的一致性以及数据库的一致性，也就是解决**丢失更新、脏读、不可重复读、幻读、序列化异常的问题**. 
>
> 并发控制模型有：
>
> - 基于锁的并发控制(LBCC)
> - 基于多版本的并发控制（MVCC）
> - 封锁、时间戳、乐观并发控制(OCC)
> - 悲观并发控制（PCC）

## 1 基于锁的并发控制(悲观)

> 为了解决并发问题，数据库引入"锁"的概念。基本的封锁类型有两种：排它锁(Exclusive locks，X锁)，共享锁(Share locks,S 锁)

- 排他锁:被加锁的对象只能被持有锁的事务读取和修改，其他事务无法在该对象上加其他锁，也不能读取和修改该对象
- 共享锁:被加锁的对象可以被持事务读取，但是不能被修改，其他事务也可以在上面再加共享锁

封锁对象可以是逻辑单元和物理单元。

- 逻辑单元:属性值、属性值的集合、元组、关系、索引项、整个索引项，甚至整个数据库
- 物理单元: 页(数据页或索引页)、物理记录等

> 由于采用了封锁策略，一次只能执行一个事务，所有只会产生串行调度，迫使事务只能等待前面的事务结束之后才可以开始，所以基于锁的并发控制机制导致性能地下，并发程度低

## 2 基于多版本的并发控制(乐观)

> 因为锁机制是一种预防性的机制，读会阻塞写，写也会阻塞读，当封锁粒度较大，时间较长时并发性能久不会太好。
>
> 而 MVCC 是一种后验性的机制，读不阻塞写，写也不阻塞读，当提交的时候才验证是否有冲突，大大提升了并发性能，常见的数据库 Oracle、PostgreSQL、MySQL都是用 MVCC 并发控制机制。**MVCC 通过保存数据在某个时间点的快照，并控制元组的可见性来实现。保证一个事务无论运行多长时间，同一个事务看到一致的数据**

PostgreSQL 为每一个事务分配一个递增的、类型为 int32 的整型数作为唯一的事务 ID，称为 xid。**创建一个快照时，收集当前正在执行的事务 id 和已提交的最大事务 id。PostgreSQL 还在系统里的每一行记录上都存储了事务相关的信息，用来判断某一行记录对于当前事务是否可见**

在 PostgreSQL 的内部数据结构中，每个元组(行记录)有 4 个与事务可见性相关的隐藏列，分别是 `xmin、xmax、cmin、cmax`，其中 ** 分别是插入和删除该元组的命令在事务中的命令标识，xmin、xmax 与事务对其他事务的可见性相关**，用于同一事务中的可见性相关，用于同一个事务中的可见性判断。

```sql
mydb=# select xmin,xmax,cmin,cmax,id,ival from tbl_mvcc where id = 1;
 xmin | xmax | cmin | cmax | id | ival
------+------+------+------+----+------
  678 |    0 |    0 |    0 |  1 |    1
(1 row)
```

> 其中 xmin 保存了创建该行数据的事务的 xid，
>
> xmax 保存的是删除该行的 xid，
>
> PostgreSQL 在不同事务时间是用 **xmin 和 xmax 控制事务对其他事务的可见性**

### 1 通过 xmin 决定插入事务的可见性

> 当插入一行数据时，PostgreSQL 会将插入这行数据的事务的 xid 存储在 xmin 中。通过 xmin 值判断事务中插入的行记录对其他事务的可见性有两种情况:

**1) 由回滚的事务或未提交的事务创建的行对于任何其他事务都是不可见的。例如**

```sql
mydb=# begin;
BEGIN
-- 查看当前事务 id
mydb=# select txid_current();
 txid_current
--------------
          688
(1 row)

mydb=# insert into tbl_mvcc values(9,9);
INSERT 0 1
-- xmin：xid 689 事务可见
mydb=# select xmin,xmax,cmin,cmax,id,ival from tbl_mvcc where id = 9;
 xmin | xmax | cmin | cmax | id | ival
------+------+------+------+----+------
  689 |    0 |    0 |    0 |  9 |    9
(1 row)
```

另一个事务操作

```sql
mydb=# begin;
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          690
(1 row)
-- 由于第一个事务并没有提交，所以第一个事务对第二个事务是不可见的
mydb=# select * from tbl_mvcc where id = 9;
 id | ival
----+------
(0 rows)

end;
COMMIT
```



**2)无论提交成功或回滚的事务,xid 都会递增。**

> 对于 Repeatable Read 和 Serializable 隔离级别的事务，如果它的 xid 小于另一个事务的 xid(也就是先来的),那么另外一个事务对这个事务是不可见的

```sql
mydb=# begin transaction isolation level repeatable read;
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          692
(1 row)

```

```sql
mydb=# begin;
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          693
(1 row)

mydb=# insert into tbl_mvcc values(11,11);
INSERT 0 1
mydb=# select xmin,xmax,cmax,id,ival from tbl_mvcc where id = 11;
 xmin | xmax  | cmax | id | ival
------+------+------+------+----
  693 |    0  |    0 | 11 |   11
(1 row)
mydb=# commit;
COMMIT
```

第一个 xid 682 事务查询 id = 11 的

```sql
mydb=# select xmin,xmax,cmax,id,ival from tbl_mvcc where id =11;
 xmin | xmax | cmax | id | ival
------+------+------+----+------
(0 rows)
```

###  2 通过 xmax 决定更新删除事务的可见性

xmax 判断事务的更新操作和删除操作对其他事务的可见性有这几种情况:

**1)如果没有设置 xmax 值，该行对其他事务总是可见的**

**2)如果它被设置为回滚事务的 xid，该行对其他事务也是可见的**

**3)如果它被设置为一个正在运行，没有 commit 和 rollback 的事务 xid，该行对其他事务是可见的；**（怎么决的这里是不可见的）

**4)如果它被设置为一个已提交的事务 xid，该行对在这个已提交事务之后发起的所有事务都是不可见的**（怎么决的这里是可见的）



:sos:这四个都不是很理解，跟自己尝试得到效果不一样，也不是很理解什么叫没有设置 xmax 值，这个不是当执行修改或删除自动设置的吗？

```sql
-- 1. 创建第一个事务并查看 xmin、xmax
mydb=# begin;
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          696
(1 row)
mydb=# select xmin,xmax,id,ival from tbl_mvcc where id = 11;
 xmin | xmax | id | ival
------+------+----+------
  694 |    0 | 11 |  110
(1 row)

-- 2. 开启第二个事务，并查看 xmin、xmax
mydb-# begin;
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          697
(1 row)
mydb=# select xmin,xmax,id,ival from tbl_mvcc where id = 11;
 xmin | xmax | id | ival
------+------+----+------
  694 |    0 | 11 |  110
(1 row)

-- 3. 更新第一个事务并查看 xmin、xmax
mydb=# update tbl_mvcc set ival=120 where id = 11;
UPDATE 1
mydb=# select xmin,xmax,id,ival from tbl_mvcc where id = 11;
 xmin | xmax | id | ival
------+------+----+------
  696 |    0 | 11 |  120
(1 row)

-- 4. 更新后第二个事务查看 xmin、xmax 
-- 这里发现事务1修改数据并未可见，但可以从 xmax 中看出，最大可见性 xid 696
mydb=# select xmin,xmax,id,ival from tbl_mvcc where id = 11;
 xmin | xmax | id | ival
------+------+----+------
  694 |  696 | 11 |  110
(1 row)
```

### 3 通过 pageinspect 观察 MVCC

```shell
mydb=# create extension pageinspect;
CREATE EXTENSION
mydb=# \dx+ pageinspect
                Objects in extension "pageinspect"
                        Object description
-------------------------------------------------------------------
 function brin_metapage_info(bytea)
 function brin_page_items(bytea,regclass)
 function brin_page_type(bytea)
 function brin_revmap_data(bytea)
 function bt_metap(text)
 function bt_page_items(bytea)
 function bt_page_items(text,integer)
 function bt_page_stats(text,integer)
 function fsm_page_contents(bytea)
 function get_raw_page(text,integer)
 function get_raw_page(text,text,integer)
 function gin_leafpage_items(bytea)
 function gin_metapage_info(bytea)
 function gin_page_opaque_info(bytea)
 function hash_bitmap_info(regclass,bigint)
 function hash_metapage_info(bytea)
 function hash_page_items(bytea)
 function hash_page_stats(bytea)
 function hash_page_type(bytea)
 function heap_page_item_attrs(bytea,regclass)
 function heap_page_item_attrs(bytea,regclass,boolean)
 function heap_page_items(bytea)
 function page_checksum(bytea,integer)
 function page_header(bytea)
 function tuple_data_split(oid,bytea,integer,integer,text)
 function tuple_data_split(oid,bytea,integer,integer,text,boolean)
(26 rows)
```

> 下面介绍两个会函数
>
> `get_raw_page(relname text,fork text,blkno int)`和它的一个重载 `get_raw_page(relname text,blkno int)`用于读取 relation 中指定的块的值
>
> - fork 默认值是 main。main 表示数据文件的主文件
> - vm 是可见性映射的块文件
> - fsm 为 free space map 的块文件
> - init 是初始化的块
>
> get_raw_page 以一个bytea 值的形式返回一个拷贝。
>
> `heap_page_items(bytea)` 显示一个堆页面上所有的行指针。
>
> 对那些使用中的行指针，**元组头部**和**元组原始数据**也会被显示。
>
> 一般使用 `get_raw_page`函数获得堆页面映射面映射作为参数传递给 heap_page_items

创建如下试图以便更清晰地观察 PostgreSQL 的MVCC

```sql
mydb=# drop view if existes v_pageinspect;
mydb=# create view v_pageinspect AS
select '(0,' || lp ||')' AS ctid,
case lp_flags
when 0 then 'Unused'
when 1 then 'Normal'
when 2 then 'Redirect to' || lp_off
when 3 then 'Dead'
end,
t_xmin::text::int8 AS xmin,
t_xmax::text::int8 AS xmax,
t_ctid
from heap_page_items( get_raw_page('tbl_mvcc',0))
order by lp;
CREATE VIEW
```

当 insert 数据时，事务会将 insert 的数据的xmin的值设置为当前事务的xid，xmax设置为null，如下所示:

```sql
mydb=# begin;
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          704
(1 row)

mydb=# insert into tbl_mvcc values(13,13);
INSERT 0 1
mydb=# select * from v_pageinspect;
  ctid  |  case  | xmin | xmax | t_ctid
--------+--------+------+------+--------
 (0,1)  | Normal |  678 |    0 | (0,1)
 (0,2)  | Normal |  682 |    0 | (0,2)
 (0,3)  | Normal |  682 |    0 | (0,3)
 (0,4)  | Normal |  683 |  684 | (0,4)
 (0,5)  | Normal |  685 |    0 | (0,5)
 (0,6)  | Normal |  686 |    0 | (0,6)
 (0,7)  | Normal |  687 |    0 | (0,7)
 (0,8)  | Normal |  688 |    0 | (0,8)
 (0,9)  | Normal |  689 |    0 | (0,9)
 (0,10) | Normal |  691 |    0 | (0,10)
 (0,11) | Normal |  693 |  694 | (0,12)
 (0,12) | Normal |  694 |  696 | (0,13)
 (0,13) | Normal |  696 |    0 | (0,13)
 (0,14) | Normal |  699 |    0 | (0,14)
 (0,15) | Normal |  700 |    0 | (0,15)
 (0,16) | Normal |  704 |    0 | (0,16)
(16 rows)
```

当 delete 数据时，将 xmax 的值设置为当前事务的 xid，

```sql
mydb=# begin;
WARNING:  there is already a transaction in progress
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          705
(1 row)

mydb=# delete from tbl_mvcc where id = 1;
DELETE 1
mydb=# select * from v_pageinspect;
  ctid  |  case  | xmin | xmax | t_ctid
--------+--------+------+------+--------
 (0,1)  | Normal |  678 |  705 | (0,1)
 (0,2)  | Normal |  682 |    0 | (0,2)
 (0,3)  | Normal |  682 |    0 | (0,3)
 (0,4)  | Normal |  683 |  684 | (0,4)
 (0,5)  | Normal |  685 |    0 | (0,5)
 (0,6)  | Normal |  686 |    0 | (0,6)
 (0,7)  | Normal |  687 |    0 | (0,7)
 (0,8)  | Normal |  688 |    0 | (0,8)
 (0,9)  | Normal |  689 |    0 | (0,9)
 (0,10) | Normal |  691 |    0 | (0,10)
 (0,11) | Normal |  693 |  694 | (0,12)
 (0,12) | Normal |  694 |  696 | (0,13)
 (0,13) | Normal |  696 |    0 | (0,13)
 (0,14) | Normal |  699 |    0 | (0,14)
 (0,15) | Normal |  700 |    0 | (0,15)
 (0,16) | Normal |  704 |    0 | (0,16)
 (0,17) | Normal |  705 |    0 | (0,17)
(17 rows)
mydb=# end;
COMMIT
```

当 update 数据时，对于每个更新的行，**首先 delete 原先的行，再执行 insert**

```sql
mydb=# begin;
BEGIN
mydb=# select txid_current();
 txid_current
--------------
          707
(1 row)
mydb=# update tbl_mvcc set ival = 20 where id = 1;
UPDATE 1
mydb=# select * from v_pageinspect;
  ctid  |  case  | xmin | xmax | t_ctid
--------+--------+------+------+--------
 (0,1)  | Normal |  678 |  705 | (0,1)
 (0,2)  | Normal |  682 |    0 | (0,2)
 (0,3)  | Normal |  682 |    0 | (0,3)
 (0,4)  | Normal |  683 |  684 | (0,4)
 (0,5)  | Normal |  685 |    0 | (0,5)
 (0,6)  | Normal |  686 |    0 | (0,6)
 (0,7)  | Normal |  687 |    0 | (0,7)
 (0,8)  | Normal |  688 |    0 | (0,8)
 (0,9)  | Normal |  689 |    0 | (0,9)
 (0,10) | Normal |  691 |    0 | (0,10)
 (0,11) | Normal |  693 |  694 | (0,12)
 (0,12) | Normal |  694 |  696 | (0,13)
 (0,13) | Normal |  696 |    0 | (0,13)
 (0,14) | Normal |  699 |    0 | (0,14)
 (0,15) | Normal |  700 |    0 | (0,15)
 (0,16) | Normal |  704 |    0 | (0,16)
 (0,17) | Normal |  705 |    0 | (0,17)
 (0,18) | Normal |  706 |  707 | (0,19) 
 (0,19) | Normal |  707 |    0 | (0,19)
 mydb=# end;
COMMIT
```

后两条显示说明。先删除原先的行，然后再执行insert

### 4 使用 pg_repack 解决表膨胀问题

> 尽管 PostgreSQL 的 MVCC 读写不阻塞写，写不阻塞读，实现高性能和高吞吐量，但也有它不足的地方。PostgreSQL 中数据采用**堆表保存**，并且 MVCC 的旧版本和新版本存储在同一个地方，**如果更新大量数据，将会导致数据表的膨胀。**

使用 `VACUUM` 命令或者 `autovacuum` 进程将旧版本的磁盘空间标记为可用，虽然 vacuum 很高效，**但没有办法把利用的磁盘空间释放给操作系统** vacuum full 命令可以回收可用的磁盘空间，但它会阻塞所有其他的操作。

> pg_repack 是一个可以在线重建表和索引的扩展。它会在数据库中建立一个和需要清理的目标表一样的临时表，目标表中的数据 COPY 到临时表，并在临时表替换目标表。

```sql
yum install -y pg_repack10
```

> :sos:这块待定吧，现在用的 mac os 系统，后面从来一遍到 Liunx

### 5 支持事务的 DDL

> PostgreSQL 能够通过预写日志设计来执行事务的 DDL。也就是把 DDL 语句放到一个事务中，比如创建表、TRUNCATE 表等。举个创建表的例子；

```sql
mydb=# begin;
BEGIN
mydb=# create table tbl_test(ival int);
CREATE TABLE
mydb=# insert into tbl_test values(1);
INSERT 0 1
rollback;
ROLLBACK
mydb=# select * from tbl_test;
ERROR:  relation "tbl_test" does not exist


mydb=# select count(*) from tbl_mvcc;
 count
-------
    11
(1 row)

mydb=# begin;
BEGIN
mydb=# truncate tbl_mvcc;
TRUNCATE TABLE
mydb=# rollback;
ROLLBACK
mydb=# select count(*) from tbl_mvcc;
 count
-------
    11
(1 row)
```

> 🤔：`书上说 truncate 命令放在了一个事务中，但最后这个事务回滚了，表中的数据完好无损`... 感觉和上面的例子没有关系，第一例子，创建表DDL 语句放到了事务中，回滚后，表不见了。 
>
> 第二个例子？？？

# 相关问题

## 1 有时操作事务，会出现下面错误信息

```sql
ERROR:  current transaction is aborted, 
commands ignored until end of transaction block

```

> 貌似发现如果在事务中，执行了错误的操作，将会报这个错误



# 相关链接