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

## 1. 拷贝数据文件方式部署流复制

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
wal_leve= replica 
# enables archiving;off,on,or always 是否启用个归档
archive_mode = on 
# command to use to archive a logfile segment
archive_command ='/bin/date'
# max number of walsemder processes 最大 WAL 发送进程
max_wal_senders = 10 
# in logfiel segments, 16 MB each; 0 disables/
wal_keep_segements = 512

```

