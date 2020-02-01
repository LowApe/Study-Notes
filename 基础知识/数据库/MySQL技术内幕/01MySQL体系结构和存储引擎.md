# MySQL体系结构和存储引擎

## 1.定义数据库和数据库实例

- 数据库：物理操作系统文件或其他形式文件类型的集合
- 实例：MySQL 数据库由**后台线程**以及一个**共享内存区**组成。

> MySQL 被设计为一个**但进程多线程**架构的数据库

使用`ps -ef | grep mysqld`查看**进程信息**

```
ps -ef |grep mysqld
root      3211     1  0 00:08 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/iZuf6aytwpcvzkwiisgaxaZ.pid
mysql     3400  3211  0 00:08 ?        00:00:18 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=iZuf6aytwpcvzkwiisgaxaZ.err --pid-file=/usr/local/mysql/data/iZuf6aytwpcvzkwiisgaxaZ.pid --socket=/tmp/mysql.sock --port=3306
root      4744  4727  0 13:10 pts/0    00:00:00 grep --color=auto mysqld
```

mysql 3400 进程，该进程就是 MySQL 实例。该实例通过**mysqld_safe** 命令来启动数据库

> 当启动实例时，MySQL 数据库会去读取**配置文件**，根据配置文件的参数来启动数据库实例.MySQL 根据**默认参数读取顺序**获取配置信息，可以通过下面命令

```shell
mysql --help |grep my.cnf
order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf
/etc/mysql/my.cnf 
/usr/local/mysql/etc/my.cnf 
/usr/local/mysql/my.cnf ~/.my.cnf
```

>  顺序从上向下读取,如果不同文件拥有相同参数，则**读取的最后一个配置文件的参数**为准

配置文件中有一个参数 datadir ,该参数指定数据库所在的路径。



## 2.MySQL体系结构

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhly1gbcc86n0eij30hg0gojs6.jpg)



MySQL 官方体系结构

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhly1gbcgqbpg1cj31ow146e81.jpg)

从图中将 MySQL 由一下几部分组成:

- 连接池组件
- 管理服务和工具组件
- SQL 接口组件
- 查询分析器组件
- 优化器组件
- 缓冲(Cache)组件
- 插件式存储引擎
- 物理文件

## 3.MySQL存储引擎

MySQL 插件式体系结构，在应对不同的存储结构

- MySQL自带存储引擎
- 用户可自定义存储引擎

| 存储引擎          | 支持功能                                                     | 特点                                                         | 设计目标                                             |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------- |
| InnoDB 存储引擎   | 事务                                                         | 行锁设计、支持外键                                           | 在线事务处理OLTP的应用                               |
| MyISAM存储引擎    | 支持全文检索                                                 | 缓冲池只缓存索引文件，而**不缓冲数据文件**                   | OLAP数据库应用                                       |
| NDB集群引擎       | 集群                                                         | 数据全部放在**内存**                                         | 集群存储引擎                                         |
| Memory存储引擎    | 中间结果集大于 Memory 存储引擎容量，会转为 MyISAM 引擎表存在磁盘中 | 数据存放在内存，只支持表锁，默认使用哈希索引                 | 存储查询的中间结果集                                 |
| Archive存储引擎   | 仅支持 INSERT 和 SELECT 操作                                 | 行锁，是用zlib 算法进行压缩存储                              | 存储归档数据；如日志信息，主要提供高速插入和压缩功能 |
| Federated存储引擎 | 指向一台远程MySQL的默认存储引擎，仅支持MySQl 数据表          |                                                              | 类似于连接服务器和网关                               |
| Maria存储引擎     | 支持缓存数据和索引文件                                       | 行锁设计、支事务和非事务安全的选项，以及 BLOB 的字符类型的处理性能 | 替代 MyISAM                                          |
| ...               | ...                                                          | ...                                                          | ...                                                  |

**查看mysql 支持的存储引擎**

```mysql
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

**修改存储引擎**

```mysql
mysql> alter table ramp Engine=InnoDB;
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

## 4.连接 MySQL

| 连接方式           | 命令                                                         |
| ------------------ | ------------------------------------------------------------ |
| TCP/IP             | `mysql -h ip -u root -p`                                     |
| 命名管道和共享内存 | 两个通信的进程在**同一个服务器上使用**<br />在 MySQL 配置文件添加`-enable-named-pipe`选项开启命名管道<br />在 MySQL 配置文件添加 `--shared-memory`开启共享内存，连接时使用`-protocol=memory` |
| UNIX 域套接字      | 暂无法进入                                                   |

