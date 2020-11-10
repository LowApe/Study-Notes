# 分区表

> PostgreSQL 10 版本一个重量级的新特性是支持内置分区表，目前支持范围分区和列表分区。内容
>
> - 内置分区表
> - 分区表的创建、性能测试和注意点

# 分区表的意义

- 当查询或更新一个分区上的大部分数据时，对分区进行索引扫描代价很大，然而，在分区上使用顺序扫描能**提升性能**
- 当需要删除一个分区数据时，通过 drop table 删除一个分区，远比 delete 删除数据高效，特别适用于日志数据场景
- 由于一个表只能存储在一个表空间上，使用分区表后，可以将分区放到不同的表空间上，**例如，可以讲系统很少访问的分区放到廉价的存储设备上，也可以将系统常访问的分区存储在高速存储上**

# 传统分区表

> 传统分区表是通过继承和触发器方式实现的，需要定义父表、定义子表、定义子表约束、创建子表索引、创建分区插入、删除、修改函数和触发器等。

## 1 继承表

首先定义一张父表，之后可以创建子表并继承(inherits)父类

```sql
mydb=# create table tbl_log(id int4,create_date date,log_type text);
CREATE TABLE
```

之后创建一种子表 `tbl_log_sql`用于存储 SQL 日志

```sql
mydb=# create table tbl_log_sql(sql text) inherits(tbl_log);
CREATE TABLE
mydb=# \d tbl_log_sql
               Table "public.tbl_log_sql"
   Column    |  Type   | Collation | Nullable | Default
-------------+---------+-----------+----------+---------
 id          | integer |           |          |
 create_date | date    |           |          |
 log_type    | text    |           |          |
 sql         | text    |           |          |
Inherits: tbl_log
```

从上面表结构看出 `tbl_log_sql`表有四个字段,前三个字段和父表 tbl_log 一样，第四个字段为自己的 sql。父表和子表都可以插入数据，接着分别在父表和子表中插入一条数据

```sql
mydb=# insert into tbl_log values(1,'2017-08-26',null);
INSERT 0 1
mydb=# insert into tbl_log_sql values(2,'2017-08-27',null,'select 2');
INSERT 0 1
```

如果查询父表会显示两表的记录，其中一条是子类记录，但是并没有将子类 sql 字段显示，可以通过 SQL 查看表的 OID确认来源

```sql
mydb=# select * from tbl_log;
 id | create_date | log_type
----+-------------+----------
  1 | 2017-08-26  |
  2 | 2017-08-27  |
(2 rows)

mydb=# select tableoid,* from tbl_log;
 tableoid | id | create_date | log_type
----------+----+-------------+----------
    16394 |  1 | 2017-08-26  |
    16400 |  2 | 2017-08-27  |
(2 rows)

mydb=# select p.relname,c.* from tbl_log c,pg_class p where c.tableoid = p.oid;
   relname   | id | create_date | log_type
-------------+----+-------------+----------
 tbl_log     |  1 | 2017-08-26  |
 tbl_log_sql |  2 | 2017-08-27  |
(2 rows)
```

如果仅查询父表名称前加上关键字 ONLY

```sql
mydb=# select * from only tbl_log;
 id | create_date | log_type
----+-------------+----------
  1 | 2017-08-26  |
(1 row)
```

> ⚠️ update、delete、select 操作，如果父表名称前没有加 ONLY，则会对父表和所有子表进行 DML 操作

```sql
mydb=# delete from tbl_log;
DELETE 2
mydb=# select count(*) from tbl_log;
 count
-------
     0
(1 row)
```

## 2 创建分区表

传统分区表创建：

1. 创建父表，如果父表上定义了约束，子表会继承，因此**除非是全局约束，否则不应该在父表上定义约束，另外，父表不应该写入数据**
2. 通过 inherits 方式创建继承表，也称为子表或分区，子表的字段定义应该和父表保持一致
3. 给所有子表创建约束，只要满足约束条件的数据才能写入对应分区，注意分区约束值范围不要有重叠
4. 给所有子表创建索引，由于继承操作不会继承父表上的索引
5. 在父表上定义 insert、delete、update触发器，将SQL分发到对应分区，【可选】，因为应用可以根据分区规则定位到对应分区进行DML操作
6. 启用 constraint_exclusion 参数，如果这个参数设置 off，父表sql性能会降低

示例演示创建一张范围分区表

```sql
-- 创建父表
 create table log_ins(id serial,user_id int4,create_time timestamp(0) without time zone);
CREATE TABLE
-- 创建 13 张子表
create table log_ins_history(CHECK (create_time < '2017-01-01')) inherits(log_ins);
...
-- 给子表创建索引
create index idx_his_ctime On log_ons_history USING btree (create_time);
..
-- 创建触发器函数，设置数据插入父表时的路有规则
create or replace function log_ins_insert_trigger();
...

```

## 3 使用分区表



## 4 查询父表还是子表

查询计划

```sql
mydb= explain analyze 
```

## 5 constaint_exclusion 参数



## 6 添加分区



## 7 删除分区



## 8 分区表相关查询



## 9 性能测试



## 10 传统分区表注意事项



# 内置分区表

