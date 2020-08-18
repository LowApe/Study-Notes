# 数据类型

关键词

- 常规和非常规数据类型
- 数据类型操作符和函数
- 数据类型转换



# 数字类型

## 1. 数字类型列表

PostgreSQL 支持的数字类型

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1ghvc5t9weoj30w80ja14v.jpg)

smallint、integer、bigint 都是**整数类型**，存储一定范围将会报错。分别字段可定义为 int2、int4、int8

```sql
-- 创建 integer 类型的表
create table test_integer(id1 integer,id2 in4);
```

deciamal 和 numeric 是等效的，存储指定**精度的多位数据**

```sql
-- 声明语法
numberic(precision,scale)
-- precision 指数字里的全部位数
-- scale 指小数部分的数字位数
-- 如 18.123 precision 5 scale 3
```

real 和 double precision 是指**浮点数据类型**，real 支持 4 字节，double precision 支持 8 字节，浮点数据类型在实际生产案例的使用相比整数类型会少些



smallserial 、 serial 和 bigserial 类型指**自增 serial 类型**，严格不能称为一种数据类型

```sql
mydb=# create table test_serial(id serial,flag text);
CREATE TABLE
Time: 6.827 ms
mydb=# \d test_serial
                            Table "public.test_serial"
 Column |  Type   | Collation | Nullable |                 Default
--------+---------+-----------+----------+-----------------------------------------
 id     | integer |           | not null | nextval('test_serial_id_seq'::regclass)
 flag   | text    |           |          |
```

```sql
-- 自动插入
insert into test_serial(flag) values('a');
insert into test_serial(flag) values('b');
insert into test_serial(flag) values('c');
select * from test_serial;
 id | flag
----+------
  1 | a
  2 | b
  3 | c
(1 row)
```



## 2. 数字类型操作符和数学函数



# 相关问题



# 相关链接