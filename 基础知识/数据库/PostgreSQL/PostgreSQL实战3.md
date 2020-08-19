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

PostgreSQL 支持数字类型操作符和丰富的数学函数

```sql
mydb=# select 1+2,2*3,4/2,8%3; -- 加减乘除取余
 ?column? | ?column? | ?column? | ?column?
----------+----------+----------+----------
        3 |        6 |        2 |        2
(1 row)

-- 数学函数 取余
mydb=# select mod(8,3)
mydb-# ;
 mod
-----
   2
(1 row)
-- 四舍五入
mydb=# select round(10.2),round(10.6);
 round | round
-------+-------
    10 |    11
(1 row)
-- 向上取整 ceil() 向下取整 floor()
mydb=# select ceil(10.2),ceil(-10.2);
 ceil | ceil
------+------
   11 |  -10
(1 row)
```



# 字符类型

## 1. 字符类型列表

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1ghwhwh464bj30vg07oq6b.jpg)

- character varying(n) 变长字符类型，超过 n 报错(？那还叫变长。。。)

- character(n) 定长字符类型，**不够长度时会用空白填充，浪费空间**
- text 任意长度的字符串

```sql
mydb=# create table test_cahr(col1 varchar(4),col2 character(4));
CREATE TABLE
mydb=# insert into test_cahr(col1,col2) values('a','a');
INSERT 0 1
-- 范围内字符串长度都为 1
mydb=# select char_length(col1),char_length(col2) from test_cahr;
 char_length | char_length
-------------+-------------
           1 |           1
(1 row)

-- 查看两字段实际占用的物理空间大小
mydb=# select octet_length(col1),octet_length(col2) from test_cahr;
 octet_length | octet_length
--------------+--------------
            1 |            4
(1 row)
```

> character varying 不声明长度，将存储**任意长度的字符串**
>
> 而 character(n) 不声明长度则等效于长度为 1

## 2. 字符类型函数

```sql
-- 长度
mydb=# select char_length('asdasd')
mydb-# ;
 char_length
-------------
           6
(1 row)

-- 占用字节数
mydb=# select octet_length('abcd');
 octet_length
--------------
            4
(1 row)

-- 指定字符在字符串的位置
mydb=# select position('a' in 'nasds');
 position
----------
        2
(1 row)

-- 提取字符串中的子串
mydb=# select substring('farar',3,4);
 substring
-----------
 rar
(1 row)

-- 拆分字符串
mydb=# select split_part('sdad,ada,dasda,',',',2);
 split_part
------------
 ada
(1 row)
```



# 时间/日期类型





# 相关问题



# 相关链接