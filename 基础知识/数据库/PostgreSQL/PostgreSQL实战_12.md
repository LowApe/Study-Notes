# 物理复制和逻辑复制

通过流复制技术，可以从实例级复制出一个与主库一摸一样的从库。

物理复制：流复制，是基于实例级的复制，只能复制整个 PostgreSQL示例

逻辑复制：也称为选择性复制，逻辑复制可以做到 **基于表级别的复制**

WAL：Write-Ahead Logging 日志记录数据库的变化，格式为二进制格式，主机断电，数据没来得及刷新数据文件，当数据库再次启动时会根据 WAL日志文件进行事务回滚，从而恢复数据库到一致性状态。**流复制是基于 WAL 物理复制，逻辑复制是基于 WAL 逻辑解析，将 WAL 解析成一种清晰、易于理解的格式**

**流复制和逻辑复制主要差异**

- 流复制是物理复制，其核心原理是主库将预写日志 WAL 日志流发送给备库，备考接收到 WAL 日志流后进行重做，因此**流复制是基于 WAL 日志文件的物理复制** 。逻辑复制核心原理也是基于 WAL，逻辑复制会根据预先设置好的规则解析WAL 日志，将 WAL 二进制文件解析成一定格式的逻辑变化信息，比如从 WAL 中解析指定表上发生的 DML 逻辑变化信息，之后主库将逻辑变化信息发送给备考，备考收到 WAL 逻辑解析信息后再应用日志
- 流复制只能对 PostgreSQL 实例级进行复制，而逻辑复制能对数据库表级进行复制。
- **流复制能对 DDL 操作进行复制，比如主库上新建表、给已有表加减字段时会自动同步到备库，而逻辑复制主库上的DDL操作不会复制到备库**
- 流复制主库可读写，但从库只允许查询不允许写入，而逻辑复制的从库可读写
- 流复制要求 PostgreSQL 大版本必须一致，逻辑复制支持跨 PostgreSQL 大版本

# 1 异步流复制

异步流复制是指主库上提交事务时不需要等待备库接收 WAL 日志流并写入到备库 WAL日志流便返回成功，而同步流复制相反

异步流复制部署方式：

- 拷贝数据文件方式
- 通过`pg_basebackup`命令行工具

## 1 拷贝数据文件方式部署流复制

装备两台主机或虚拟机，分别作为主节点和备件点

> 🤔：如果只有一台电脑，能不能使用 docker 进行模拟，待验证

```shell
# 创建操作系统用户和相关目录
groupadd postgres
useradd postgres -g postgres
passwd postgres
# pg_root 存储数据库系统数据文件
mkdir -p /database/pg10/pg_root
# pg_tbs 存储用户自定义表空间文件
mkdir -p /database/pg10/pg_tbs
chown -R postgres:postgres /databases/pg10
```

流复制配置参数`postgresql.conf`,设置一下参数:

```properties
# minial ,replica , or logical WAL 日志输出级别
wal_level = replica 
# enables archiving;off,on,or always 是否启用个归档,
archive_mode = on 
# command to use to archive a logfile segment
archive_command ='/bin/date'
# max number of walsemder processes 最大 WAL 发送进程
max_wal_senders = 10 
# in logfiel segments, 16 MB each; 0 disables/
wal_keep_segements = 512
# 控制数据库恢复过程中是否启动读操作，这个参数通常用在流复制备库
# 开启参数后流复制 备库支持只读SQL，但备库不支持写操作，主库上也设置此参数为 on
hot_standby
```

> 以上是流复制配置过程中主要的 `postgresql.conf`参数，其他参数没有列出，主库和备库的 `postgresql.conf` 配置建议完全一致

配置主库的 `pg_hba.conf` 文件，然后启动

```properties
# replication privilege
host replication repuser ip1/32 md5
host replication repuser ip2/32 md5
```

使用超级用户 postgres 登陆到数据库创建流复制用户 repuser，流复制用户需要有 `replication`权限和 `login`权限🤔：需要单独创建一个数据库用户吗？

```sql
create user repuser
replication
login
connection limt 5
encrpted password 'xxxxx'
```

**热备生成一个备份，制作备份过程中主库仍然可读写，不影响主库上的业务,以postgres 超级用户执行**

```sql
postgres=# select pg_start_backup('frances_bk1');
 pg_start_backup
-----------------
 0/46000028
(1 row)
```

> 待定ing 生成文件找不到，书上居然也都不写，直接复制？？

## 2 以 pg_basebackup 方式部署流复制

部署流复制备库的数据复制环节主要包含三个步骤。

1. pg_stat_backup('francs_bk1');
2. 拷贝主节点`$PGDATA`数据文件和表空间文件到备节点；
3. pg_stop_backup()

以上三个步骤可以合成一步完成，PostgreSQL 提供内置的 `pg_basebackup`命令行工具支持对主库发起一个在线基准备份，备份完成后自动从备份模式退出。**pg_basebackup工具是对数据库实例级进行的物理备份，因此这个工具通常作为备份工具进行基准备份**

```shell
pg_basebackup -D
/root/PostgreSQL/database/pg12/pg_root 
-Fp -Xs -v -P -h ip1 -p 1921 -U repuser
```

> 上面命令备库使用 pg_basebackup 命令对主库做基准备份
>
> pg_basebackup 命令首先对数据库做一次 checkpoint ,之后基于时间点做一个全库基准备份，**全备过程中会拷贝$PGDATA数据文件和表空间文件到备库节点对应目录**，pg_basebackup 参数：
>
> - -D 指定备节点用来接收主库数据的目标路径，这里和主库保持一致，依然是`/database/pg10/pg_root`目录
> - -F 参数指定 `pg_basebackup`命令生成的备份数据格式，支持两种格式，p格式和 t 格式,p 格式是指生成的备份数据和主库上的数据文件布局一样，t 格式将备份文件打个 tar 包并存储在指定目录里，系统文件被打包成 base.tar,其他表空间文件被打包成 oid.tar，其中 OID 为表空间的 OID
> - -X 参数设置在备份的过程中产生的WAL 日志包含在备份方式，f(fetch)和 s(stream), f 是指WAL 日志在基准备份完成后被传送到备节点，这时主库上的 `wal_keep_segments`参数需要设置得较大，以免备份过程中产生的 WAL 还没发送到备节点之前被主库覆盖掉，这样会失败，f方式下主库将会启动一个基准备份 WAL 发送进程；s 方式中主库上除了启动一个基准备份 WAL 发送进程外还会额外启动一个 WAL 发送进程用于发送主库产生的 WAL 增量日志流
> - -v 启动 verbose 模式，打印各阶段日志
> - -P 显示数据文件、表空间文件近似传输百分比，由于执行 pg_basebackup命令过程中主库数据文件会变化，因此这只是一个估算值
> - -h -p -U 远程地址 端口号 用户

pg_basebackup 命令执行成功后，配置备库 `recovery.conf`,之前已将文件备份到home？？？，从 home 目录复制到 $PGDATA 目录

```shell
cp ~/revocery.conf $PGDATA
# 启动备库
pg_ctl start
```

这时备库节点已经有了 WAL 接收进程，同时主节点已经有了 WAL 发送进程🤔怎么查看的



## 3 查看流复制同步方式

postgresql 内置的表`pg_stat_replication`



# 2 同步流复制



# 3 单实例、异步流复制、同步流复制性能测试



# 4 流复制监控

流复制部署完成之后，通常需要监控流复制主库、备库的状态

## 1 pg_stat_replication

主库上主要监控 WAL 发送进程信息，`pg_stat_replication`试图显示 WAL 发送进程的详细信息，这个视图对于流复制的监控非常重要。表字段的字段：

| 字段             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| pid              | WAL 发送进程的进程号                                         |
| usename          | WAL 发送进程的数据库用户名                                   |
| application_name | 连接 WAL 发送进程的应用别名，此参数显示值为备考<br />recovery.conf 配置文件中 primary_conninfo 参数 <br />application_name 选项的值 |
| client_addr      | 连接到 WAL 发送进程的客户端 IP 地址，也就是**备库的 Ip**     |
| backend_start    | WAL 发送进程的启动时间                                       |
| state            | 显示 WAL 发送进程的状态，<br />startup 表示 WAL 进程在启动过程中<br />catchup 表示备库正在追赶主库<br />streaming 表示备库已经追赶上了主库<br />,并且主库向备库发送 WAL 日志流，这个状态是流复制的常规状态<br />backup 表示通过 pg_basebackup 正在进行备份<br />stopping 表示 WAL 发送进程正在关闭 |
| sent_lsn         | WAL 发送进程最近发送的 WAL 日志位置                          |
| write_lsn        | 备库最近写入的 WAL 日志位置，这时 WAL 日志流还在操作系统缓存中，还没写入备库 WAL 日志文件 |
| flush_lsn        | 备库最近写入的 WAL 日志位置，这时 WAL 日志流已写入备库 WAL 日志文件 |
| replay_lsn       | 备库最近应用的 WAL 日志位置                                  |
| write_lag        | 主库上 WAL 日志落盘后等待备库接受 WAL 日志                   |
| flush_lag        |                                                              |
| replay_lag       |                                                              |
| sync_priority    | 基于优先级的模式中备库被选中成为同步备库的优先级             |
| sync_state       | 同步状态<br />async 表示备库为异步同步模式<br />potential 表示备库当前为异步同步模式，同步备库宕机,异步备库可升级为同步备库🤔<br />sync 表示当前备库为同步模式<br />quorum 表示备库为 quorum standbys 的候选 |

> `write_lag、flush_lag、replay_lag`三个字段为 `PostgreSQL10`新特性，**是衡量主备延迟的重要指标**

## 2 监控主备延迟

# 10 逻辑复制

PostgreSQL10 另一个重量级新特性为 **逻辑复制(Logical Replication)**

> 流复制是基于实例级别的复制，相当于主库的一个热备，也就是说备库的数据库对象和主库一摸一样，而**逻辑复制是基于表级别的选择性复制，例如可以复制主库一部分表到备库**，这是一种粒度更细的复制，逻辑复制主要使用场景为：
>
> - 根据业务需求，将一个数据库中的一部分同步到另一个数据库
> - 满足报表库取数需求，从多个数据库采集报告数据
> - 实现 PostgreSQL 跨大版本同步
> - 实现 PostgreSQL 大版本升级

流复制是基于 WAL 日志的物理复制，其原理是主库不间断地发送 WAL 日志流到备库，备库接受主库发送的 WAL 日志流后应用 WAL；

而逻辑复制是基于逻辑解析(logical decoding),其核心原理是主库将 WAL 日志流解析成一定格式，**订阅节点收到解析的WAL数据流后进行应用，从而实现数据同步**，逻辑复制并不是使用 WAL 原始日志文件进行复制，而是将 WAL 日志解析成了一定格式

## 1 逻辑解析

逻辑解析(logical decoding)是逻辑复制的核心，理解逻辑解析有助于理解逻辑复制原理，逻辑解析读取数据库的 WAL 并将数据变化解析成目标格式，这一小节将对逻辑解析进行演示。

逻辑解析的前提是设置 wal_level 参数为 logical 并且设置 max_replication_slots 参数至少为 1，如下所示：

```properties
wal_level = logical
# 允许的最大复制槽数 8
max_replication_slots = 8
```

pghost1 数据库上创建逻辑复制槽,如下所示：

```sql
-- 创建 逻辑复制槽
select pg_create_logical_replication_slot('logical_slot1','test_decoding');
-- 查看 逻辑复制槽
select * from pg_replication_slots;
```

之前物理复制槽的主要作用是避免主库可能覆盖并循环使用还没有发给备库的 WAL 日志。

```sql
-- 查看逻辑复制槽解析的数据变化
select * from pg_logical_slot_get_changes('logical_slot1',null,null);
```

`pg_logical_slot_get_changes`函数用来查看指定逻辑复制槽所解析的数据变化。每执行一次，所解析的数据变化被消费掉，也就是说查询结果不能复现.测试逻辑复制槽是否可以捕获 DDL 数据，先创建一个测试表，再通过上面查看数据变化。

因为此函数捕获的数据将被消费掉，因此，此函数查询结果仅能显现一次。

> `select * from pg_logical_slot_peek_changes('logical_slot1',null,null); 函数用来查看指定逻辑复制槽所解析重现`

如果逻辑复制槽不需要使用了，需要及时删除：

```sql
select pg_drop_replication_slot('logical_slot1');
```

## 2 逻辑复制框架

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gmga21470qj30vc0dktdb.jpg)

> 逻辑复制是基于逻辑解析，其核心原理是逻辑主库将 `Publication`中表的WAL日志解析成一定格式并发送给逻辑备库，逻辑备库`Subscription`接受到解析的 WAL 日志后进行重做，从而实现表数据同步

逻辑复制架构图中最重要的两个角色为 `Publication`和`Subscription`。

- `Publication（发布）`可以定义在任何可读写 `PostgreSQL`实例上，对于已创建 `Publication`的数据库称为发布节点，**一个数据库中允许创建多个发布，目前允许将多个表注册到一个发布中**加入发布的表通常需要有复制标识(replicaidentity),从而使逻辑主库表上的`DELETE/UPDATE`操作可以标记到相应数据行并复制到逻辑备库上的相应表，默认情况下使用主键作为复制标识，如果没有主键也可以是唯一索引，可设置复制标识为 full,意思是整行数据作为键值，这种情况下复制效率会降低。如果加入发布的表没有指定复制标识，表上的 `UPDATE/DELETE`将会报错
- `Subscription（订阅）`实时同步指定发布者的表数据，位于逻辑复制的下游节点，对于已创建`Subscription`的数据库称为订阅节点，订阅节点的数据同时也能创建发布。**发布节点上发布的表的 DDL 不会被复制，因此，如果发布节点上发布的表结构更改了，订阅节点上需手工对订阅的表进行 DDL 操作，订阅节点通过逻辑复制槽获取发布节点发送的 WAL 数据变化**

## 3 逻辑复制部署

准备两个 postgreSQL 示例

发布节点的 `postgresql.conf`配置文件设置参数：

```properties
# 支持逻辑复制
wal_level = logical
# 设置大于订阅节点的数量
max_replication_slots = 8
max_wal_senders = 10
```

订阅节点 `postgresql.conf`配置文件设置参数：

```properties
# 设置数据库复制槽数量，应大于订阅节点的数量
max_replication_slots = 8
# 设置逻辑复制进程数，应大于订阅节点的数量，并且给表同步预留一些进程数量，默认4
max_logical_replication_workers = 8

max_worker_processes
```

`max_logical_replication_workers`会消耗后台进程数，并且从 `max_worker_processes`参数设置的后台进程数中消费，因此 max_worker_processes 参数需要设置较大



发布节点上创建逻辑复制用户，逻辑复制用户需要具备 `REPLICATION`权限，如下所示：

```sql
CREATE USER logical_user
    REPLICATION
    LOGIN
    CONNECTION LIMIT 8
    ENCRYPTED password 'logical_user';
```

**创建发布的语法**

```sql
CREATE PUBLICATION name
	[FOR TABLE[ONLY] table_name [*][,...]
      | FOR ALL TABLES ]
      [WITH ( publication_parameter [= value][,...])]
	
```

- name: 指发布的名称。
- FOR TABLE: 指加入到发布的表列表，目前仅支持普通表的发布，临时表、外部表、视图、物化视图、分区表暂不支持发布，如果想将分区表添加到发布中，需逐个添加分区表分区到发布
- FOR ALL TABLES：将当前库中所有表添加到发布中，包括以后在这个库中新建的表。这种模式相当于在全库级别逻辑复制所有表。当然一个 `PostgreSQL`实例上可以运行多个数据库，这任然是仅复制了 PostgreSQL 实例上的一部分数据

如果想查询刚创建的发布信息，在发布节点上查询 `pg_publication`视图即可，字段：

- pubname: 指发布的名称
- pubbowner: 指发布的属主，和`pg_user`视图的 `usesysid`字段关联
- puballtables: 是否发布数据库中的所有表，t 表示发布数据库中所有已存在的表和以后新建的表
- pubinsert: `t`表示仅发布表上的 INSERT 操作
- pubudate: `t`表示仅发布表上的 UPDATE 操作
- pubdelete: `t`表示仅发布者上的 DELETE 操作

之后计划在订阅节点上创建订阅，语法如下：

```sql
CREATE SUBSCRIPTION subscription_name
	CONNECTION 'conninfo'
	PUBLICATION publication_name [,...]
	[ WITH (subscription_parameter)]
```

- subsription_name: 指订阅的名称
- conncetion: 订阅的数据库连接串，通常包括 host、port、dbname、user、password 等连接属性，从安全角度考虑，密码文件建议写入`~/.pgpass`隐藏文件
- publication：指定需要订阅的发布名称
- with `(subscription_parameter[=value][,...])`支持的参数配置有 `copy_data(boolean)、create_slot(boolean)、enabled (boolean)、slot_name(string)`等，一般默认配置即可

订阅节点上查看 `pg_subscription`视图以查看订阅信息

```sql
select * from pg_subscription;
```

视图字段：

- subdbid：数据库的 OID，和 pg_database.oid 关联
- subname: 订阅的名称
- subowner： 订阅的属主
- subenabled：是否启用订阅
- subconninfo: 订阅的连接串信息，显示发布节点连接串信息
- subslotname: 复制槽名称
- subpublications: 订阅节点订阅的发布列表

> 注意⚠️：订阅完成，有可能无法初始化复制数据，可能没有对 pguser 模式的读权限,需要对逻辑复制用户授权
>
> ```sql
> GRANT USAGE ON SCHEMA pguser TO logical_user;
> GRANT SELECT ON t_lr1 TO logical_user;
> ```

## 4 逻辑复制 DML 数据验证

验证发布节点的 `INSERT/UPDATE/DELETE`操作是否会同步到订阅节点。



> 虽然订阅节点上的表 `t_lr1`能够实时同步发布节点上 `t_lr1`表上的数据，实际上订阅节点上的 `t_lr1`表也支持写操作，只是对订阅节点逻辑复制的表进行写操作时有可能与逻辑同步产生冲突，导致逻辑复制停止，冲突详细信息可通过订阅节点的数据库日志查看



## 5 逻辑复制添加表、删除表

实际生产维护过程中有增加逻辑同步表的需求，逻辑复制支持向发布中添加同步表，并且操作非常方便。

将表 `t_big` 的 `SELECT` 的权限赋给逻辑复制用户 `logical_user`。

```sql
GRANT SELECT ON t_big TO logical_user;
```

 之后在发布节点上将表`t_big`加入到发布pub1,如下所示：

```java
ALTER PUBLICATION pub1 ADD TABLE t_big;
```

查看发布中的表列表，执行`\dRp+`元命令即可；

```sql
mydb=> \dRp+ pub1
```

也可以通过查看 `pg_publication_tables`视图查看发布中的表列表

```sql
select * from pg_publication_tables;
```

由于 `t_big`表是发布节点上新增加的表，这里订阅节点上 `t_big`表的数据还没有复制过来，订阅节点需要执行一下命令：

```SQL
ALTER SUBSCRIPTION sub1 REFERSH PUBLICATION
```

**这条命令执行之后，订阅节点的`t_big`表在同步发布节点的数据了，同时在发布节点主机上产生了逻辑复制 COPY 发送进程，大概 33 秒左右**

由于需求调整，逻辑复制中 `t_big`表不再需要逻辑同步，只需要在发布节点上将 `t_big`表从发布 pub1 中去掉即可，执行如下命令：

```sql
ALTER PUBLICATION pub1 DROP TABLE t_big;
```

## 6 逻辑复制启动、停止

逻辑复制配置完成之后，默认情况下订阅节点的表会实时同步发布节点中的表，逻辑复制通过启用、停止订阅方式实现逻辑复制的启动和停止。

 **订阅节点上停止 sub1 订阅从而中断实时同步数据：**

```sql
ALTER SUBSCRIPTION sub1 DISABLE;
```

查询 `pg_subscription` 视图的 subenabled 字段判断是否已停止订阅，如下所示：

```sql
select * from pg_subscription
```

**如果想开启订阅，执行如下命令：**

```sql
ALTER SUBSCRIPTION sub1 ENABLE;
```

## 7 逻辑复制配置注意事项和限制

发布节点配置注意事项如下：

- 发布节点的 `wal_level`参数需要设置成 `logical`
- 发布节点上逻辑复制用户至少需要 `replication`角色权限
- 发布节点上需要发布的表如果需要将 UPDATE/DELETE 操作同步到订阅节点，需要给发布表配置复制标识
- 发布时可以选择发布 INSERT、UPDATE、DELETE DML 操作中的一项或多项，默认是发布这三项
- 支持一次发布一个数据库中的所有表
- 一个数据库中可以有多个发布
- 逻辑复制目前仅支持普通表，序列、视图、物化视图、分区表、外部表等对象目前不支持
- 发布节点配置文件 `pg_hba.conf`需做相应配置，允许订阅节点连接
- 发布表上的 DDL 操作不会自动同步到订阅节点，如果发布节点上发布的表执行了 DDL 操作，需手动给订阅节点的相应表执行 DDL



订阅节点配置注意事项如下：

- 一个数据库中可以有多个订阅
- 必须具有超级用户权限才可以创建订阅
- 创建订阅时需指定发布节点连接信息和发布名称
- 创建订阅时默认不会创建发布节点的表，因此创建订阅前需手工创建表
- 订阅支持启动、停止操作
- 订阅节点的表结构建议和发布节点一致，尽管订阅节点的表允许有额外的字段
- 发布节点给发布增加表时，订阅节点需要刷新订阅才能同步新增的表



对于配置了复制标识的表，`UPDATE/DELETE` 操作可以逻辑复制到订阅节点，但目前的版本中逻辑复制有以下限制：

- DDL 操作不支持复制，发布节点上发布表进行 DDL 操作后，DDL 操作不会复制到订阅节点，需在订阅节点对发布表手工执行 DDL 操作
- 序列本身不支持复制，当前逻辑复制仅支持普通表，序列、视图、物化视图、分区表、外部表等对象都不支持
- TRUNCATE 操作不支持复制
- 大对象(Large Object) 字段不支持复制

## 8 逻辑复制延迟测试

测试逻辑复制的延迟



