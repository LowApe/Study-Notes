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