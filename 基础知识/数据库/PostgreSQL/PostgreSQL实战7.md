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

## 1 基于锁的并发控制



# 相关链接