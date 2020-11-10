# 性能优化

在硬件层面，影响数据库性能的主要因素有 CPU、I/O、内存和网络；

在软件层面则要复杂得多，操作系统配置、中间件配置、数据库参数配置、运行在数据库之上的查询和命令等

本节内容：

- 一些关键的性能指标、判断性能瓶颈的方法
- 常见的性能检测和系统监控工具
- 常见的性能瓶颈的解决思路

# 1 服务器硬件

影响数据性能的主要硬件因素有 CPU、磁盘、内存和网络。

**磁盘**

最先到达瓶颈的，通常是磁盘的I/O,投入生产之前应该对磁盘的容量和吞吐量进行估算，磁盘容量，做好扩展准备。如果读写密集的生产环境使用廉价的消费级SSd。**使用外部存储设备加载到服务器也是比较常见的，例如 SAN(存储区域网络)和 NAS (网络接入存储)**  SAN 设备时通过块接口访问，对服务器来说就像访问本地磁盘一样；NAS则是使用标准文件协议进行访问，例如 NFS 等。**使用外部存储，还需要考虑网络通信对响应时间的影响**

**CPU**

CPU 也会经常称为性能瓶颈，数据库服务器执行的每一个检查都会给 CPU 增加一定压力。数据库服务器禁用节能模式

**内存**

内存的使用对数据库系统非常重要。

# 2 操作系统优化

数据库是与操作系统结合非常紧密的系统应用，操作系统的参数配置会直接作用在数据库服务器。

## 1 常用 Linux 性能工具

> top、free、vmstat、iostat、mpstat、sar、pidstat等。除了 top 和 free 外，其余工具均位于 sysstat 包中。`yum install -y sysstat`

**top**

top 命令是最常用的性能分析工具，它可以实时监控系统状态，输出系统整体资源占用状况以及各个进程的资源占用状况。



**free**

free 命令显示当前系统的内存使用情况，

```shell
root@iZuf6aytwpcvzkwiisgaxaZ:~# free
              total        used        free      shared  buff/cache   available
Mem:        2048212      326552       89232      155976     1632428     1388292
Swap:             0           0           0
```

Linux 为了提升 I/O 性能和减小磁盘压力，使用了 buffer cache 和 page cache，buffer cache 针对磁盘的写进行缓存，直接对磁盘进行操作的数据会缓存到 buffer cache，而系统文件中的数据则是交给 page cache，



**vmstat**

```shell
# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0  89824  19272 1613404    0    0    50   288  292  678  1  1 98  0  0
```

vmstat 是 Linux 中的虚拟内存统计工具，用于监控操作系统虚拟内存、进程、CPU等整体情况。vmstat 最常规的用法是:`vmstat delay count`,即每隔delay 采样一次，共采样 count 次，如果省略 count

🤔

**iostat**

iostat 命令用于整个系统、适配器、tty 设备、磁盘和 CD-ROM 的输入/输出统计信息，最常用的是用 iostat 来监控磁盘的输入输出

```shell
# iostat
Linux 4.4.0-93-generic (iZuf6aytwpcvzkwiisgaxaZ) 	11/10/2020 	_x86_64_	(1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.21    0.00    0.84    0.20    0.00   97.75

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               3.85        49.62       283.49     739741    4225996

```

**mpstat**

mpstat 返回 CPU 的详细性能信息

```shell
# mpstat 5 5
Linux 4.4.0-93-generic (iZuf6aytwpcvzkwiisgaxaZ) 	11/10/2020 	_x86_64_	(1 CPU)
08:32:31 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
08:32:36 PM  all    0.81    0.00    0.60    0.00    0.00    0.00    0.00    0.00    0.00   98.59
08:32:41 PM  all    0.61    0.00    0.41    0.00    0.00    0.00    0.00    0.00    0.00   98.99
Average:     all    0.73    0.00    0.41    0.04    0.00    0.04    0.00    0.00    0.00   98.78
```

在 mpstat 的最后一行会有一个运行期间的平均统计值，默认的 mpstat 会统计所有 CPU 的信息，如果只需要观察某一个 CPU，加上参数 -P n，n 为要观察的 core 的索引。如果观察 CPU 0 的统计信息，每5秒采样一次，共采样 3 次`mpstat -P 0 5 3`

**sar**

sar 是性能统计非常的工具

**其他性能工具的方法**

nmon,像一个图形界面的 top



## 2 Linux 系统的 I/O调度算法





## 3 预读参数调整



## 4 内存的优化



# 3 数据库调优

## 1 全局参数调整

在 `postgresql.conf`配置文件中有很多参数可以灵活配置数据库的行为，对性能影响较大的参数的调整方法和原则.

- `shared_buffers=128m`共享内存，数据库启动时，即使没有请求，共享内存也会保持固定的大小。
- `work_mem=4mb` 用来限制每个服务进程进行排序或 hash 时的内存分配，指定内部排序和 hash 在使用临时磁盘文件之前能使用的存数量
- `random_page_cost=4`代表随机访问磁盘块的代价估计

## 2 统计信息和查询计划

在运行期间，PostgreSQL 会收集大量的数据表、表、索引的统计信息。查询优化器通过这些统计信息估计查询运行的时间，然后选择最快的查询路径。这些统计信息都保存在 PostgreSQL 的系统表中，这些系统表都有 `pg_stat`或`pg_statio`开头。决定何时运行 autovacuum 和如何解释查询计划。这些数据保存在 `pg_staistics`中，这个表只有超级用户可读，普通用户没有权限，查看这些数据，可以从 `pg_stats`视图中查询。另一类统计数据用于监测数据库级、表级、语句级的信息

**pg_stat_database**

数据库级

```sql
mydb=# \d pg_stat_database
                        View "pg_catalog.pg_stat_database"
        Column         |           Type           | Collation | Nullable | Default
-----------------------+--------------------------+-----------+----------+---------
 datid                 | oid                      |           |          |
 datname               | name                     |           |          |
 -- 当前多少个并发连接
 numbackends           | integer                  |           |          |
 xact_commit           | bigint                   |           |          |
 xact_rollback         | bigint                   |           |          |
 -- 读取磁盘块的次数与这些块的缓存命中数
 blks_read             | bigint                   |           |          |
 blks_hit              | bigint                   |           |          |
 tup_returned          | bigint                   |           |          |
 tup_fetched           | bigint                   |           |          |
 tup_inserted          | bigint                   |           |          |
 tup_updated           | bigint                   |           |          |
 tup_deleted           | bigint                   |           |          |
 conflicts             | bigint                   |           |          |
 temp_files            | bigint                   |           |          |
 temp_bytes            | bigint                   |           |          |
 deadlocks             | bigint                   |           |          |
 checksum_failures     | bigint                   |           |          |
 checksum_last_failure | timestamp with time zone |           |          |
 blk_read_time         | double precision         |           |          |
 blk_write_time        | double precision         |           |          |
 stats_reset           | timestamp with time zone |           |          |
```

**pg_stat_user_tabls**

表级的统计信息

```sql
mydb=# \d pg_stat_user_tables
                      View "pg_catalog.pg_stat_user_tables"
       Column        |           Type           | Collation | Nullable | Default
---------------------+--------------------------+-----------+----------+---------
 relid               | oid                      |           |          |
 schemaname          | name                     |           |          |
 relname             | name                     |           |          |
 seq_scan            | bigint                   |           |          |
 seq_tup_read        | bigint                   |           |          |
 idx_scan            | bigint                   |           |          |
 idx_tup_fetch       | bigint                   |           |          |
 n_tup_ins           | bigint                   |           |          |
 n_tup_upd           | bigint                   |           |          |
 n_tup_del           | bigint                   |           |          |
 n_tup_hot_upd       | bigint                   |           |          |
 n_live_tup          | bigint                   |           |          |
 n_dead_tup          | bigint                   |           |          |
 n_mod_since_analyze | bigint                   |           |          |
 last_vacuum         | timestamp with time zone |           |          |
 last_autovacuum     | timestamp with time zone |           |          |
 last_analyze        | timestamp with time zone |           |          |
 last_autoanalyze    | timestamp with time zone |           |          |
 vacuum_count        | bigint                   |           |          |
 autovacuum_count    | bigint                   |           |          |
 analyze_count       | bigint                   |           |          |
 autoanalyze_count   | bigint                   |           |          |
```

**pg_stat_statements**

语句级的统计信息一般通过`pg_stat_statements`、`postgres日志` 

开启 `pg_stat_statements`需要在 `postgresql.conf`中配置

```properties
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track =all
```

```sql
--然后执行 create extension 启用它
create extension pg_stat_satements;
```

**查看 SQL 的执行计划**

执行计划，也叫做查询计划，会显示扫描语句中用到的表，例如使用顺序扫描还是索引扫描等等，以及多个表连接时使用什么连接算法来把每个输入表的行连接在一起。在 PostgreSQL 中使用 `explain` 命令来查看执行计划，例如：

```sql
mydb=# explain select * from log_ins;
                         QUERY PLAN
------------------------------------------------------------
 Seq Scan on log_ins  (cost=0.00..28.50 rows=1850 width=16)
(1 row)
-- analyze 真实的查询计划
mydb=# explain analyze select * from log_ins;
                                              QUERY PLAN
------------------------------------------------------------------------------------------------------
 Seq Scan on log_ins  (cost=0.00..28.50 rows=1850 width=16) (actual time=0.002..0.003 rows=0 loops=1)
 Planning Time: 0.040 ms
 Execution Time: 0.015 ms
(3 rows)
```

> 待完善

## 3 索引管理与维护

