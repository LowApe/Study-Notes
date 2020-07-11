# 客户端工具

当创建了数据目录，创建、管理数据库实例，以及基础的数据库配置项。可以在一个数据库实例中创建用户以及数据库了。通常我们通过客户端工具或者命令去连接并操作数据库实例。因为客户端工具相对来说有图像化界面，操作简单，但因为底层还是通过命令方式操作数据库，所有我们这里就只介绍命令。

# psql 功能及应用

> psql 是 PostgreSQL 自带的命令行客户端工具



## 使用 psql 连接数据库

```shell
# 格式
pgsql [options...] [dbname[username]]
# 第一个是 连接数据库 第二个表示用户名
psql postgres postgres
psql (12.3)
Type "help" for help.

postgres=>
```

## 创建用户及数据库

```shell
# 创建用户
postgres=# create role testuser with encrypted password 'testuser';
CREATE ROLE
# 创建表空间目录
root$mkdir -p database/pg12/pg_tbs_/tbs_mydb
# 创建表空间
postgres=# create tablespace tbs_mydb owner testuser location '/Users/mac/Documents/AllMyDocuments/Working/02/PostgreSQL/database/pg12/pg_tbs/tbs_mydb';
CREATE TABLESPACE
# 创建数据库
postgres=# create database mydb
with owner = testuser template = template0 encoding = 'UTF8' tablespace = tbs_mydb;
CREATE DATABASE
# 赋权
postgres=# grant all on database mydb to testuser with grant option;
GRANT
postgres=# grant all on tablespace tbs_mydb to testuser;
GRANT
```

- create database 命令中的 `owner` 选择数据库的属主
- template 表示数据库模版 有 0 1两种，也能自定义数据库模版
- encoding 数据库字符集
- tablespace 表示数据库的**默认表空间**，新建的数据库对象会在该路径产生文件

```shell
# 远程连接
psql -h 目标ip -p 端口号 数据库 所属用户
```

## psql 元命令介绍

> psql 中的元命令是指以**反斜杠开头**的命令，psql 提供丰富的元命令，能够便捷地管理数据库，比如 查看数据库对象定义、查看数据库对象占用空间大小、列出数据库各种对象名称、数据导入等

```shell
# 查看表空间列表
mydb=# \db
                                               List of tablespaces
    Name    |  Owner   |                                        Location
------------+----------+-----------------------------------------------------------------------------------------
 pg_default | mac      |
 pg_global  | mac      |
 tbs_mydb   | testuser | /Users/mac/Documents/AllMyDocuments/Working/02/PostgreSQL/database/pg12/pg_tbs/tbs_mydb
(3 rows)
# 创建一张表
mydb=# create table test_1(id int4,name text,create_time timestamp without time zone default clock_timestamp());
CREATE TABLE
# 查看表定义
mydb=# \d
        List of relations
 Schema |  Name  | Type  | Owner
--------+--------+-------+-------
 public | test_1 | table | mac
(1 row)
# 添加主键
mydb=# alter table test_1 add primary key (id);
ALTER TABLE
# 查看具体表定义
mydb=# \d test_1
                                Table "public.test_1"
   Column    |            Type             | Collation | Nullable |      Default
-------------+-----------------------------+-----------+----------+-------------------
 id          | integer                     |           | not null |
 name        | text                        |           |          |
 create_time | timestamp without time zone |           |          | clock_timestamp()
Indexes:
    "test_1_pkey" PRIMARY KEY, btree (id)          |          | clock_timestamp()


```

### 查看表、索引占用空间大小

> generate_series 函数产生连续的整数，使用这个函数能非常方便地产生测试数据

```shell
# 插入 50万数据
mydb=# insert into test_1(id,name) select n,n || '_frances' from generate_series(1,500000) n;
INSERT 0 500000
# 查看表空间大小
mydb-# \dt+ test_1
                   List of relations
 Schema |  Name  | Type  | Owner | Size  | Description
--------+--------+-------+-------+-------+-------------
 public | test_1 | table | mac   | 29 MB |
(1 row)
# 查看索引空间大小
mydb-# \di+ test_1_pkey
                          List of relations
 Schema |    Name     | Type  | Owner | Table  | Size  | Description
--------+-------------+-------+-------+--------+-------+-------------
 public | test_1_pkey | index | mac   | test_1 | 11 MB |
(1 row)
```

### \sf 查看函数代码

```shell
# 内容待定，书上的看不懂
```

### \x 设置查询结果输出

```shell
# 普通查询显示
mydb=# select * from test_1 limit 1;
 id |   name    |        create_time
----+-----------+----------------------------
  1 | 1_frances | 2020-07-11 21:53:56.009174
(1 row)
# 设置查询结果输出 格式？
mydb=# \x
Expanded display is on.
mydb=# select * from test_1 limit 1;
-[ RECORD 1 ]---------------------------
id          | 1
name        | 1_frances
create_time | 2020-07-11 21:53:56.009174
```

### 获取元命令对应的SQL代码

```shell
\db
```



# 相关连接