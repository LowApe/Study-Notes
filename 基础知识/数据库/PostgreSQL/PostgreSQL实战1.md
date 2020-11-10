# ps 安装与基本配置

# 安装与基本配置

## 安装

Mac 下使用 Homebrew 下载

```shell
brew install postgresql
```

> 其他方式可以查看[官网下载安装方式](https://www.postgresql.org/download/)

## 客户端程序和服务器程序

### 目录

- bin (应用程序)
- include (PostgreSQL 的 C、C++ 的头文件)
- lib (库文件)
- share (PostgreSQL 的文档、man、示例文件一些扩展)

### 客户端

在bin 目录下:

1. 封装SQL命令的客户端程序

| 名称                   | 描述                                                         | 例子                                                         |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Clusterdb              | clusterdb 是 SQL CLUSTER 命令的一个封装。PostgreSQL 是**堆表存储的**，clusterdb通过索引对数据库中基于堆表的物理文件重新排序<br />它在一定场景下可以节省磁盘访问，加快查询速度 | `clusterdb -h pghost1 -p 1921 -d mydb`                       |
| reindexdb              | Reindexed 是SQL REDINDEX 命令的一个封装。在索引物理**文件发生损坏或索引膨胀**等状况发生时，可以使用 reindexdb 命令对指定的**表或者数据库重建索引并且删除旧的索引** | `reindexdb -e -h pghost1 -p 1921 -d mydb`                    |
| vacunmdb               | Vacuumed 是 PostgreSQL 数据库独有的VACUUM、VACUUM FREEZE 和 VACUUM FULL，VACUUM ANALYZE 这几个 SQL 命令的封装。<br />VACUUM系列命令的主要职责是对**数据的物理文件等的垃圾回收，是 PostgreSQL 中非常重要的一系列命令** | `vacuumb -h pghost1 -p 1921 mydb`                            |
| vacuumlo               | vacuumlo 用来清理数据库中未引用的大对象                      | `vacuumlo -h pghost1 -p 1921 mydb`                           |
| Created 和 dropdb      | 分别是 SQL 命令 CREATE DATABAS 和 DROPD ATABASE的封装        | `dropdb -h pghost1 -p 1921 newdb`                            |
| createuser 和 dropuser | 它们分别是 SQL 命令 CREATE USER 和 DROP USER 的封装。<br />创建一个newuser 非超级用户，没有创建数据库权限，没有创建用户的权限，并且立即给它设置密码 | `createuser -h pghost1 -p 1921 -c 1 -g pg_monitor --interactive (-D -R -S -P) -e newuser`<br />删除用户`dropuser -h pghost1 -p 1921 newuser` |

### 备份与恢复的客户端程序

- `pg_basebackup` 取得一个正在运行中的 PostgreSQL 实例的基础备份。

- `pg_dump` 和 `pg_dumpall` 都是以数据库转存方式进行备份的工具
- `pg_restore`用来从 `pg_dump` 命令创建的非文本格式的备份中恢复数据

### 其他客户端程序

- `ecpg`是用于 C 程序的 PostgreSQL 嵌入式 SQL 预处理器。它将 SQL 调用替替换为特殊函数调用。
- `oid2name`解析一个 PostgreSQL 数据目录中的 OID 和文件结点
- `pgbeach` 是运行基准测试的工具，平常我们可以用它模拟简单的**压力测试**。
- `pg_config`获取当前安装的 PostgreSQL 应用程序的配置参数
- PostgreSQL 包装了 `pg_isready` 工具用来检测数据库服务器是否已经允许接受连接。
- `pg_receivexlog`可以从一个运行中的实例获取事务日志的流
- `pg_recvlogical` 控制逻辑解码复制槽以及来自这种复制槽的流数据
- `psql` 是连接 PostgreSQL 数据库的客户端命令行工具，是使用频率非常高的工具 `psql -h pghost1 -p 1921 mydb`

### 服务器程序

服务器程序包括：

- `initdb`用来创建新的数据库目录
- `pg_archivecleanup`是清理PostgreSQL WAL 归档文件的工具
- `pg_controldata`显示数据库服务器的控制信息，例如目录版本、预习日志和检查点的信息
- `pg_ctl`是初始化、启动、停止、控制数据库服务器的工具
- `pg_resetwal`可以清除预写日志并且有选择地重置存储在 `pg_control`文件中的一些控制信息
- `pg_rewind`是在 master、salve 角色发生切换时，将原 master 通过同步模式恢复，避免重做基础备份的工具
- `pg_test_fsync`可以通过一个快速的测试，了解系统使用哪一种预写日志的同步方法(wal_sync_method)最快
- `pg_test_timing`是一种度量系统计时开销以及确认系统时间绝不会回退的工具
- `pg_upgrade`是PostgreSQL 的升级工具
- `pg_waldump`用来将预写日志解析为可读的格式
- `postgres`是 PostgreSQL 的服务器程序
- `postmaster`可以从 bin 目录看到，是指向 postgre 服务器程序的一个软链接`postmaster -> postgres`

# 创建数据库实例

在 PostgreSQL 中有一个**数据库实例**和一组使用**相同配置文件**和**监听端口的数据库集**关联，它由**数据目录**组成，数据目录中包含了所有的数据文件和配置文件。

- 一台数据库服务器可以管理多个数据库实例，PostgreSQL 通过数据目录的位置和这个数据集合实例的端口号引用它。

## 创建数据目录

```shell
mkdir -p /pgdata/12/{data,backups,scripts,archive_wals}
```



## 创建操作系统用户

在**创建数据库实例**之前第一件事先创建一个独立的操作系统用户，也可以称为本地用户。

> 创建这个账号的目的是为了防止因为引用软件的 BUG 被攻击者利用，对系统造成破坏，**坚持使用非管理员账号运行 PostgreSQL **. yum 安装会自动创建不存在的 postgres 本地用户和 postgres 的数据库。**一定要创建非管理员账号，并且切换用户，然后在创建数据目录**

> 之前一直创建不上，之后重启又可以初始化了，也可能是因为initdb 的数据目录没有开启权限

```shell
chown -R 用户:用户组 pgdata/12
chmod 0700 /pgdata/12/data
```

> initdb 会回收除 PostgreSQL 用户之外所有用户访问权限。但是我们有明确的数据库存储目录，所以使用上述命令保护。因为我是 mac 系统经过测试，发现下载完后初始化默认使用管理员身份当作用户名，我也没有进行用户切换，文件夹授权也是给了当前登陆用户

## 初始化数据目录

`initdb --help` 查看相关参数命令

```shell
# -W 参数，所有在初始化的过程中,initdb 工具会要求为数据库超级用户创建密码
initdb -D pgdata/12/data -W
The files belonging to this database system will be owned by user "mac".
This user must also own the server process.

The database cluster will be initialized with locale "zh_CN.UTF-8".
The default database encoding has accordingly been set to "UTF8".
initdb: could not find suitable text search configuration for locale "zh_CN.UTF-8"
The default text search configuration will be set to "simple".

Data page checksums are disabled.

Enter new superuser password: 
Enter it again: 

fixing permissions on existing directory pgdata/12/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Asia/Shanghai
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    pg_ctl -D pgdata/12/data -l logfile start
```

> 除了使用initdb 来初始化数据目录，还可以使用 `pg_ctl`工具进行数据库目录的初始化`pg_ctl init -D /pgdata/10/data -o "-W"`。使用 yum 安装会自动创建`/var/lib/pgssql/12`目录和两个子目录



# 启动和停止数据库服务器

根据上面初始化最后的提示启动服务器

```shell
# 启动项目
pg_ctl -D pgdata/12/data -l logfile start
# 查看状态
pg_ctl -D pgdata/12/data status
# 检测数据库服务器是否已经允许接受连接
pg_isready -p 1921
# 停止数据库 支持三种停止 smart fast immediate
pg_ctl -D pgdata/12/data -ms stop
```

其他启动和停止数据库的方式，例如使用 `postmaster`或者 `postgres` 程序启动数据库

```shell
# 添加 & 让后台运行
postgres -D pgdata/12/data &
```

## 配置开机启动

> mac 设置待定，linux 貌似有一系列固定的命令，因为shell 那本书还没看完，这块待定

# 数据库配置基础

- 全局配置：在一个数据库实例中，有些配置会影响到整个实例。全局配置文件
	- `postgresql.conf` 负责文件位置、资源限制、集群复制等
	- `pg_hba.conf`客户端连接和认证
- 非全局配置：有些配置只对一个数据库实例中的单个 Database 生效或者某个数据库用户生效

> 两个文件都在初始化数据目录中

## pg_hba.conf

它是所在数据库实例的 “防火墙”：作用是设置允许那些主机可以通过什么**连接方式和认证方式**通过哪个数据库用户连接哪个数据库。

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1ggbr3c2u1sj31280cwmzd.jpg)

| 名称               | 详情                                                         |
| ------------------ | ------------------------------------------------------------ |
| Type连接方式       | local匹配使用` Unix` 域套接字的连接。<br />host匹配使用`TCP/IP`建立的连接，同时匹配 `SSL` 和非`SSL`连接,不允许使用`TCP/IP`远程连接，启用远程连接需要修改`postgresql.conf`的`listen_addresses`参数。<br />`hostssl`必须是使用 SSL 的`TCP/IP`连接。匹配`hostssl`有三个前提条件:<br />1.客户端和服务端都安装 `OpenSSL`<br />2.编译`PostgreSQL`的时侯指定`configure参数 --with-openssl 打开SSL支持`<br />3.在 `postgresql.conf中配置ssl=on` |
| DataBase目标数据库 | DATABASE 列标识该行设置对哪个数据库生效                      |
| USER目标用户       | 该行设置对哪个数据库用户生效                                 |
| ADDRESS访问来源    | 标识该行设置对哪个 IP 地址或 IP地址段生效                    |
| METHOD认证方式     | METHOD 列标识客户端的认证方法，常见的认证方法有`trust、reject、md5和password`等<br />1.reject 认证方式场景：允许某一网段的大多数主机访问数据库，但**拒绝这一网段的少数特定主机**<br />2.md5 和 password 认证方式的区别md5双重加密，password 指明文密码，所以不要在**非信任网络**使用 password 认证方式 |

## postgresql.conf

> PostgreSQL.conf 配置文件，由多个 configparameter = value 形式的行组成,"#"开头的行为注释 value 支持的数据类型有布尔、整数、浮点数、字符串、枚举，value的值还支持各种单位,例如 MB、GB 和 ms、min等 当配置项末尾标记了`#(change requires restart)`的配置项是需要重启数据库实例才可以生效的，其他没有标记的配置项只需要 reload 即可生效

| 名称               | 解释                                                         |
| ------------------ | ------------------------------------------------------------ |
| 全局配置的修改方法 | `mydb=# ALTER SYSTEM SET listen_addresses = '*'`，这个在数据库命令修改配置自动编辑`postgresql.auto.conf`文件，在数据库启动时会加载该文件，并用它的配置覆盖 `postgresql.conf`中已有的配置。**这个文件不要手动修改它** |
| 非全局配置修改     | 设置和重置 DataBase级别<br />`ALTER DATABASE name SET configparameter {TO | =}{value| DEFAULT}` <br />`ALTER DATABASE name RESET configuration`<br /> |
| 查看配置           | 查询 `pg_settings 系统表 select * from pg_settings`          |
| 使配置生效         | `pg_ctl -D /pgdata/12/data reload`                           |

## 允许远程访问数据库

- **修改监听地址**(conf文件`listen_addresses='*'` 允许所有，然后重启，然后还需要配置`pg_hba.conf`允许通过什么连接方式和认证方式通过哪个数据库用户连接)

```properties
# - Connection Settings -

#listen_addresses = 'localhost'		# what IP address(es) to listen on;
					# comma-separated list of addresses;
					# defaults to 'localhost'; use '*' for all
					# (change requires restart)
```

假设我们允许所有主机通过 TCP/IP 建立的连接，同时匹配 SSL 和 非 SSL 连接，通过 md5 口令认证，使用`pguser`用户

```shell
# 向 pg_hba.conf 文件中添加一行
echo "host mydb pguser 0.0.0.0/0 md5" >> 
/pgdata/12/data/pg_hba.conf
# 修改后 reload 生效
pg_ctl -D /pgdata/12/data/ restart
```



# 相关问题

## 初始化错误

```shell
initdb:directory "/pgdata/12/data" exist but is not empty
```

需要删除对应文件夹文件以及隐藏文件



## 启动错误

```shell
# 启动
pg_ctl -D pgdata/12/data -l logfile start
waiting for server to start.... stopped waiting
pg_ctl: could not start server
Examine the log output.
```

查看了 logfile 文件，说是绑定地址失败，我又使用Postgres.app 去启动，出现端口号报错，然后我就去查看进程和端口号 `ps -ef | grep postgres`和 `lsof -i:5432`，发现已经开启服务和占用了端口号。

> 然后我回想起 mac 使用 `brew install postgresql`下载安装，并且启动，默认帮我初始化数据库目录到 `/usr/loacl/var/postgres`，因为是按照教程上来，所以结束服务`brew services postgresql`,使用上面启动命令启动，就成功了。 

# 相关链接

- [Download](https://www.postgresql.org/download/)
- [官网](www.postgresql.org)
- [mac上brew安装pg数据库默认没有创建postgres用户?](https://blog.csdn.net/kmust20093211/article/details/47104175?utm_source=blogxgwz2)