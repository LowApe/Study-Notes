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



## 时间/日期类型列表

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gi281aaxxbj30wo0b6n52.jpg)



```sql
-- 系统当前时间
mydb=# select now();
              now
-------------------------------
 2020-08-24 21:29:37.240428+08
(1 row)

-- 类型转换
mydb=# select now()::time without time zone;
       now
-----------------
 21:34:36.597064
(1 row)

-- 时间精度
mydb=# select now(),now()::timestamp(0);
              now              |         now
-------------------------------+---------------------
 2020-08-24 21:36:28.950915+08 | 2020-08-24 21:36:29
(1 row)
```

> 默认时间类型精度为 6

## 时间/日期类型操作符

```sql
-- 日期相减

mydb=# select date '2019-12-12' - interval'1 hour';
      ?column?
---------------------
 2019-12-11 23:00:00
(1 row)
```

```sql
-- 日期相乘
mydb=# select 100* interval '1 second';
 ?column?
----------
 00:01:40
(1 row)

-- 日期相除

mydb=# select interval '1 hour'/ double precision '3';
 ?column?
----------
 00:20:00
(1 row)
```

## 时间/日期类型常用函数

```sql
-- 显示当前时间
mydb=# select current_date,current_time;
 current_date |    current_time
--------------+--------------------
 2020-08-24   | 21:39:52.145902+08
(1 row)

-- 抽取函数 EXTRACT 函数 格式：EXTRACT(field FROM source)
-- field: century year month day hour minute second ...
-- source: timestamp time interval
mydb=# select extract(year from now());
 date_part
-----------
      2020
(1 row)

mydb=# select extract(month from now()),extract(day from now());
 date_part | date_part
-----------+-----------
         8 |        24
(1 row)

-- 当天属于当年的第几天

mydb=# select extract(doy from now());
 date_part
-----------
       237
(1 row)
```



# 布尔类型

> 之前数字、字符、日期是关系型数据库的常规数据类型，此外 PostgreSQL 还支持很多非常规数据类型，比如布尔类型、网络地址类型、数组类型、范围类型、json/jsonb 类型等

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gi28qhjgg5j30vw064tac.jpg)



| true状态有效值 | false状态有效值 |
| -------------- | --------------- |
| TRUE           | FASLE           |
| true           | false           |
| t              | f               |
| y              | n               |
| yes            | no              |
| on             | off             |
| 1              | 0               |

测试

```sql
create table test_boolean(cola boolean,colb boolean);
mydb=# insert into test_boolean(cola,colb) values('true','false');
INSERT 0 1
mydb=# insert into test_boolean values('t','f');
INSERT 0 1
mydb=# insert into test_boolean values('TRUE','FALSE');
INSERT 0 1
mydb=# insert into test_boolean values('yes','no');
INSERT 0 1
mydb=# insert into test_boolean values('y','n');
INSERT 0 1
mydb=# insert into test_boolean values('1','0');
INSERT 0 1
mydb=# insert into test_boolean values(null,null);
INSERT 0 1
mydb=# select * from test_boolean;
 cola | colb
------+------
 t    | f
 t    | f
 t    | f
 t    | f
 t    | f
 t    | f
      |
(7 rows)
```

# 网络类型

> 当有存储 IP 地址需求的业务场景时，一般会存储字符类型，实际上可以提供用于存储 IPv4、IPv6、MAC 网络地址的专有网络地址数据类型，使用网络地址类型存储会对数据合法性进行检查，也提供网络数据操作符和函数方便开发

## 网络地址类型列表

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gi29bnp6bkj30vc0a6adg.jpg)

```sql
-- inet 和 cidr 类型存储的网络地址格式为 address/y  ip地址/网络掩码
-- 格式校验
mydb=# select '192.168.2.1000'::inet;
ERROR:  invalid input syntax for type inet: "192.168.2.1000"
LINE 1: select '192.168.2.1000'::inet;
-- 正确的
mydb=# select '192.168.2.100'::inet;
     inet
---------------
 192.168.2.100
(1 row)
 -- cidr 带位数
mydb=# select '192.168.2.100'::cidr;
       cidr
------------------
 192.168.2.100/32
 -- 32 不显示
(1 row)
mydb=# select '192.168.2.100/32'::inet;
     inet
---------------
 192.168.2.100
(1 row)
-- 其他显示
mydb=# select '192.168.2.100/16'::inet;
       inet
------------------
 192.168.2.100/16
(1 row)
```

- cidr 会对合法性进行检查，而 inet 不会

## 网络地址操作符

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gi29kkt60nj30we0oitn5.jpg)

## 网络地址函数

```sql
-- 取 ip
mydb=# select host(cidr '192.168.1.0/24');
    host
-------------
 192.168.1.0
(1 row)
-- 取子网掩码
mydb=# select netmask(cidr '192.168.1.0/24');
    netmask
---------------
 255.255.255.0
(1 row)
-- 全都取
mydb=# select text(cidr '192.168.1.0/24');
      text
----------------
 192.168.1.0/24
(1 row)
```







# 相关问题



# 相关链接