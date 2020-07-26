# 客户端工具

当创建了数据目录，创建、管理数据库实例，以及基础的数据库配置项。可以在一个数据库实例中创建用户以及数据库了。通常我们通过客户端工具或者命令去连接并操作数据库实例。因为客户端工具相对来说有图像化界面，操作简单，但因为底层还是通过命令方式操作数据库，所有我们这里就只介绍命令。

# psql 功能及应用

> psql 是 PostgreSQL 自带的命令行客户端工具



## 使用 psql 连接数据库

```shell
# 格式
psql [options...] [dbname[username]]
# 第一个是 连接数据库 第二个表示用户名
psql postgres postgres
psql (12.3)
Type "help" for help.

postgres=>
```

远程连接

```sql
# 远程连接
psql -h 目标ip -p 端口号 数据库 所属用户
```



## psql 的参数

| 参数 | 解释                                          | 命令                                                         |
| ---- | --------------------------------------------- | ------------------------------------------------------------ |
| -A   | 非对齐模式                                    | `psql -A -c "select * from test_1 where id=1" mydb testuser;` |
| -t   | 只显示记录数据                                | `psql -t -c "select * from test_1 where id=1" mydb testuser;` |
| -q   | psql执行SQL命令会返回多种消息；不显示输出信息 | `psql -q mydb testuser -f xxx.sql`                           |

## psql 执行sql 脚本

创建一个 sql 后缀的脚本，并写sql 语句。



```shell
# 语句为：
drop test_2 if exists test_2;
create test_2(id int4);
insert into test_2 values(1);
insert into test_2 values(2);
# 执行 sql 脚本
$ psql mydb testuser  -f test.sql
DROP TABLE
CREATE TABLE
INSERT 0 1
INSERT 0 1
```

> Sql 的 `-single-trasaction`或`-1`选项支持在一个事务中执行脚本。要么都成功要么失败回滚

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
mydb=#  
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
# 带 sql代码的元命令
psql -E mydb
psql (12.3)
Type "help" for help.

mydb=# \db
********* QUERY **********
SELECT spcname AS "Name",
  pg_catalog.pg_get_userbyid(spcowner) AS "Owner",
  pg_catalog.pg_tablespace_location(oid) AS "Location"
FROM pg_catalog.pg_tablespace
ORDER BY 1;
**************************

                                               List of tablespaces
    Name    |  Owner   |                                        Location
------------+----------+-----------------------------------------------------------------------------------------
 pg_default | mac      |
 pg_global  | mac      |
 tbs_mydb   | testuser | /Users/mac/Documents/AllMyDocuments/Working/02/PostgreSQL/database/pg12/pg_tbs/tbs_mydb
(3 rows)
```

### 查看有哪些元命令

```shell
# \? 查询手册
\?
General
  \copyright             show PostgreSQL usage and distribution terms
  \crosstabview [COLUMNS] execute query and display results in crosstab
  \errverbose            show most recent error message at maximum verbosity
  \g [FILE] or ;         execute query (and send results to file or |pipe)
  \gdesc                 describe result of query, without executing it
  \gexec                 execute query, then execute each value in its result
  \gset [PREFIX]         execute query and store results in psql variables
  \gx [FILE]             as \g, but forces expanded output mode
  \q                     quit psql
  \watch [SEC]           execute query every SEC seconds

Help
  \? [commands]          show help on backslash commands
  \? options             show help on psql command-line options
  \? variables           show help on special variables
  \h [NAME]              help on syntax of SQL commands, * for all commands

Query Buffer
  \e [FILE] [LINE]       edit the query buffer (or file) with external editor
  \ef [FUNCNAME [LINE]]  edit function definition with external editor
  \ev [VIEWNAME [LINE]]  edit view definition with external editor
  \p                     show the contents of the query buffer
  \r                     reset (clear) the query buffer
  \s [FILE]              display history or save it to file
  \w FILE                write query buffer to file

```

### HELP 命令

`\h`后面接 SQL 命令关键字能将 SQL 命令的语法列出

```shell

mydb=# \h create tablespace
Command:     CREATE TABLESPACE
Description: define a new tablespace
Syntax:
CREATE TABLESPACE tablespace_name
    [ OWNER { new_owner | CURRENT_USER | SESSION_USER } ]
    LOCATION 'directory'
    [ WITH ( tablespace_option = value [, ... ] ) ]

URL: https://www.postgresql.org/docs/12/sql-createtablespace.html
```

`\h`命令后面不接任何 SQL 命令则列出所有 SQL 命令，为查看 SQL 命令语法提供参考

```shell
mydb=# \h
Available help:
  ABORT                            CREATE USER
  ALTER AGGREGATE                  CREATE USER MAPPING
  ALTER COLLATION                  CREATE VIEW
  ALTER CONVERSION                 DEALLOCATE
  ALTER DATABASE                   DECLARE
  ALTER DEFAULT PRIVILEGES         DELETE
  ALTER DOMAIN                     DISCARD
  ALTER EVENT TRIGGER              DO
  ALTER EXTENSION                  DROP ACCESS METHOD
  ALTER FOREIGN DATA WRAPPER       DROP AGGREGATE
  ALTER FOREIGN TABLE              DROP CAST
  ALTER FUNCTION                   DROP COLLATION
  ALTER GROUP                      DROP CONVERSION
  ALTER INDEX                      DROP DATABASE
  ALTER LANGUAGE                   DROP DOMAIN
  ALTER LARGE OBJECT               DROP EVENT TRIGGER
  ALTER MATERIALIZED VIEW          DROP EXTENSION
  ALTER OPERATOR                   DROP FOREIGN DATA WRAPPER
  ALTER OPERATOR CLASS             DROP FOREIGN TABLE
  ALTER OPERATOR FAMILY            DROP FUNCTION
  ALTER POLICY                     DROP GROUP
  ALTER PROCEDURE                  DROP INDEX
  ALTER PUBLICATION                DROP LANGUAGE
  ALTER ROLE                       DROP MATERIALIZED VIEW
  ALTER ROUTINE                    DROP OPERATOR
  ALTER RULE                       DROP OPERATOR CLASS
:
```

## Sql 导入、导出表数据

- `COPY` SQL 命令
- `\copy`是元命令
- `COPY`**命令必须具有 SUPERUSER 超级权限**(将数据通过 stdin、stdout 方式导入导出情况除外)，而`\copy`元命令**不需要** `SUPERUSER` 权限
- `COPY`命令读取或写入数据库服务端主机上的文件，而`\copy`元命令是从`psql`客户端主机读取或写入文件
- **从性能方面，大数据量导出到文件或大文件数据导入数据库，COPY 比 \copy 性能高**

### 使用 COPY 命令导入导出数据

**导入：**

```shell
# 创建一个copy 导入的表
postgres=# create table test_copy(id int4,name text);
CREATE TABLE
postgres=# \q
# 切换 superuser 用户进行导入
$ psql mydb mac
psql (12.3)
Type "help" for help.
mydb=# copy test_copy from '/Users/mac/Documents/AllMyDocuments/Working/02/PostgreSQL/test_copy_in.txt';
COPY 3
```

**可能出现的问题：**

下一步需要切换到superuser 否则会出现下面错误信息：

```shell
mydb=> copy test_copy from '/Users/mac/Documents/AllMyDocuments/Working/02/PostgreSQL/test_copy_in.txt';
ERROR:  must be superuser or a member of the pg_read_server_files role to COPY from a file
HINT:  Anyone can COPY to stdout or from stdin. psql's \copy command also works for anyone.
```

创建导入文本，需要用`tab`分割，不能是空格：

```shell
mydb=# copy test_copy from 'xxx/test_copy_in.txt';
ERROR:  invalid input syntax for type integer: "1 a"
CONTEXT:  COPY test_copy, line 1, column id: "1 a"
```

**导出：**

导出将表数据输出到标准输出，而且不需要超级用户权限；

```shell
mydb=>copy test_copy to stdout;
1	a
2	b
3	c
# 导出 csv 格式
copy test_copy to 'xxx/test_copy.csv' with csv header;
COPY 3
```

上述 `with csv header` 是知道出格式为 csv 格式并且显示字段名称

```shell
 # 导出部分
 mydb=>copy (select * from test_copy where id =1) to 'xxx/1.txt';
 COPY 1
```

## 使用\copy 元命令导入导出数据

```shell
 # 导出
 mydb=>\copy test_copy to 'xxxx/test_copy_2.txt';
 # 导入
 mydb=>\copy test_copy from 'xxxx/1.txt';
 COPY 1
```



## psql 如何传递变量到 SQL

### \set 元命令传递变量

```shell
mydb=> \set v_id 2
mydb=> select * from test_copy where id=:v_id;
 id | name
----+------
  2 | b
(1 row)

# 取消设置变量
mydb=>\set v_id
```

### Sql 的 -v 参数传递变量

```shell
# 创建 v_sql.sql
cat v_sql.sql
select * from test_copy where id=:v_id;
# -v 传递
$ psql -v v_id=1 mydb testuser -f v_sql.sql
 id | name
----+------
  1 | a
  1 | a
(2 rows)
```

## 使用psql定制日常维护脚本

> 通过主要介绍通过 psql 元命令定制日常维护脚本，预先将常用的数据库维护配置好，数据库排障时直接使用，从而提高效率

### 1.定制维护脚本：查询活动会话

> psql 没有带 -X 选项，psql 尝试读取和执行用户`~/.psqlrc` 启动文件中的命令，结合这个命令可以方便得**预先定制维护脚本**

```shell
# 查看SQL活动会话的 SQL
select pid,usename,datname,query,client_addr from 
pg_stat_activity where pid <> pg_backend_pid() and 
state='active' order by query;
 pid | usename | datname | query | client_addr
-----+---------+---------+-------+-------------
(0 rows)
```

- `pg_stat_activity`试图显示 `PostgreSQL`进程信息，每一个进程在视图中存在一条记录
- Pid  进程号
- usename 数据库名称
- query 显示进程最近执行的 SQL
- client_addr 是进程的客户端 IP

创建或找到 `~/.psqlrc` 文件，编辑内容如下

```sql
-- 查询活动会话
\set active_session 'select pid,usename,datname,query,client_addr from pg_stat_activity where pid <> pg_backend_pid() and state=\'active\' order by query;'
```

之后重新连接数据库，执行 `active_session`命令，**冒号后接变量名即可

```sql
mydb=# :active_session
 pid | usename | datname | query | client_addr
-----+---------+---------+-------+-------------
(0 rows)

```

> 通过以上设置，数据库排障时不需要临时手工编写查询活动会话的SQL，只需输入:active_session 即可，方便了日常维护操作。
>
> 这里我尝试通过远程连接，然后查询语句，不知道怎回事还是查出来是null。。。不清楚是不是不能识别本地的连接会话

### 1.定制维护脚本：查询等待事件

编写`\set`元命令追加到`~/.psqlrc`文件

```sql
\set wait_event 'select
pid,usename,datname,query,client_addr,wait_event_type,wait_event
from pg_stat_activity where pid <> pg_backend_pid() and wait_event is not null order by wait_event_type;'
```



## psql 亮点功能



# 相关问题



## PostgreSQL创建数据库时报错：ERROR: source database "template1" is being accessed by other users

> 场景：想删除表空间和表和数据库时，出现的一个错误。

```shell
ps -ef | grep postgre
```

查看进程，并结束相关进程。

## psql: error: could not connect to server: FATAL:  role "testuser" is not permitted to log in

```shell
postgres=# alter role testuser with login;
ALTER ROLE
```

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1ggo1trnwylj324y0dg41d.jpg)

# 相关连接

- [postgresql 表空间创建、删除](https://blog.csdn.net/weixin_34160277/article/details/93354538)
- [PostgreSQL创建数据库时报错](https://blog.csdn.net/jueshengtianya/article/details/17420287/)
- [role "testuser" is not permitted to log in](https://dba.stackexchange.com/questions/57723/superuser-is-not-permitted-to-login)
- [PostgreSql 视图](https://www.postgresql.org/docs/10/static/monitoring-stats.html#pg-stat-activity-view)

