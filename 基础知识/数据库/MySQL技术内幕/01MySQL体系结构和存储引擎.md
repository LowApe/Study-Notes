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

## 3. MySQL存储引擎

