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



# 数组类型

> PostgreSQL 支持**一维数组和多维数组**，常用的数组类型为数字类型数组和字符型数组，也支持枚举类型、复合类型数组

## 数组类型定义

```sql
-- 数组类型定义
create table test_array1(
id integer,
array_i integer[],
array_t text[]
);
```

## 数组类型值输入

```sql
-- 花括号方式
'{ val1 delim val2 delim ...}'
mydb=# select '{1,2,3}';
 ?column?
----------
 {1,2,3}
(1 row)
-- 插入数据
mydb=# insert into test_array1 values (1,'{1,2,3}','{"a","b","c"}');
INSERT 0 1
-- 显示数据
mydb=# select * from test_array1;
 id | array_i | arry_t
----+---------+---------
  1 | {1,2,3} | {a,b,c}
(1 row)
```

```sql
-- 使用 ARRAY 关键字
mydb=# select array[1,2,3];
  array
---------
 {1,2,3}
(1 row)

mydb=# create table test_array2(id integer,array_i integer[],array_y text[]);
CREATE TABLE
mydb=# insert into test_array2 values(1,array[4,5,6],array['a','b','c']);
INSERT 0 1
mydb=# select * from test_array2;
 id | array_i | array_y
----+---------+---------
  1 | {4,5,6} | {a,b,c}
(1 row)

```

## 查询数组元素

```sql
-- 下表查询数组
mydb=# select array_i[1],arry_t[3] from test_array1 where id = 1;
 array_i | arry_t
---------+--------
       1 | c
(1 row)

```

> 下表的数字就是所在数组个位置，索引从 1 开始

## 数组元素的追加、删除、更新

```sql
-- 追加 array_append(anyarray,anyelement) 末端追加一个元素
mydb=# select array_append(array[1,2,3],4);
 array_append
--------------
 {1,2,3,4}
(1 row)
-- 追加数组也可以使用操作符 ||
mydb=# select array[1,2,3] || 4;
 ?column?
-----------
 {1,2,3,4}
(1 row)
```

```sql
-- 数组元素删除 array_remove(anyarray,anyelement);
mydb=# select array[1,2,2,3],array_remove(array[1,2,2,3],2);
   array   | array_remove
-----------+--------------
 {1,2,2,3} | {1,3}
(1 row)
```



```sql
-- 数组元素的修改代码
mydb=# select * from test_array1;
 id | array_i | arry_t
----+---------+---------
  1 | {1,2,3} | {a,b,c}
(1 row)

mydb=# update test_array1 set array_i[3]=4 where id = 1;
UPDATE 1
mydb=# select * from test_array1;
 id | array_i | arry_t
----+---------+---------
  1 | {1,2,4} | {a,b,c}
(1 row)
```



## 数组操作符

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gi2sksn5jvj30v20j47gj.jpg)

```sql
-- 前两个尝试
mydb=# select array[1,2,3]<>array[1,2,4];
 ?column?
----------
 t
(1 row)

mydb=# select array[1,2,3]::int[]=array[1,2,4];
 ?column?
----------
 f
(1 row)
```

## 数组函数

```sql
mydb=# select array_append(array[1,2],3),array_remove(array[1,2],2);
 array_append | array_remove
--------------+--------------
 {1,2,3}      | {1}
(1 row)

-- 获取数组纬度
mydb=# select array_ndims(array[1,2]);
 array_ndims
-------------
           1
(1 row)
-- 获取数组长度 后面数字不清楚是什么意思(\sf 查看是返回类型)
mydb=# select array_length(array['a','b','c','d'],1);
 array_length
--------------
            4
(1 row)

-- 查看数组长度代码格式
mydb=# \sf array_length
CREATE OR REPLACE FUNCTION pg_catalog.array_length(anyarray, integer)
 RETURNS integer
 LANGUAGE internal
 IMMUTABLE PARALLEL SAFE STRICT
AS $function$array_length$function$

-- 返回数组中某个数组元素第一次出现的位置
mydb=# select array_position(array['a','b','c','d'],'d');
 array_position
----------------
              4
(1 row)
-- 数组替换函数
mydb=# select array_replace(array[1,2,5,4],5,3);
 array_replace
---------------
 {1,2,3,4}
(1 row)
-- 将数组元素输出到字符串，可以使用 array_to_string(anyarray,text[,text])
-- text[,text] 解读，将值为 null 的替换 text
mydb=# select array_to_string(array[1,2,null,3],',','10');
 array_to_string
-----------------
 1,2,10,3
(1 row)
```

> 最后一个函数 array_to_string 有些问题
>
> - \sf 无法查询到
> - 共有两个参数，但是书上的例子上面是三个...

# 范围类型

> 常见的范围类型
>
> - 日期范围类型
> - 整数范围类型
> - ...
>
> 范围类型提供丰富的操作符和函数，对于日期安排、价格范围应用场景比较适用

## 范围类型列表

| 类型      | 描述                          |
| --------- | ----------------------------- |
| int4range | integer范围类型               |
| int8range | bigint 范围类型               |
| numrange  | numeric范围类型               |
| tsrange   | 不带时区的 timestamp 范围类型 |
| tszrange  | 带时区的 timestamp 范围类型   |
| daterange | date 范围类型                 |

```sql
-- 自定义范围类型
mydb=# select int4range(1,5);
 int4range
-----------
 [1,5)
(1 row)

 mydb=# select daterange('2017-07-01','2020-07-31');
        daterange
-------------------------
 [2017-07-01,2020-07-31)
(1 row)
```

## 范围类型边界

```sql
-- 如果没有指定数据类型边界模式，指定上届为"]"
mydb=# select int4range(4,7);
 int4range
-----------
 [4,7)
(1 row)
-- 指定上下界都是包含
  mydb=# select int4range(1,3,'[]');
 int4range
-----------
 [1,4)
(1 row)
-- 指定下届包含 
mydb=# select int4range(1,3,'(]');
 int4range
-----------
 [2,4)
(1 row)
```

> HINT:  Valid values are "[]", "[)", "(]", and "()".
>
> 不清楚为什么不管怎么指定，显示都是"[ )"

## 范围类型操作符

```sql
-- 包含元素操作符
mydb=# select int4range(4,7) @>4;
 ?column?
----------
 t
(1 row)
-- 包含范围操作符
mydb=# select int4range(4,7)@>int4range(4,6);
 ?column?
----------
 t
(1 row)
-- 等于操作符
mydb=# select int4range(4,7)=int4range(4,6,'[]');
 ?column?
----------
 t
(1 row)
```

> `@>`操作符在范围数据类型中比较常用，常用来查询范围数据类型是否包含某个指定元素

## 范围类型函数

```sql
-- 取上下边界值
mydb=# select lower(int4range(1,10));
 lower
-------
     1
(1 row)

mydb=# select upper(int4range(1,10));
 upper
-------
    10
(1 row)
-- 范围是否为 null ？
mydb=# select isempty(int4range(1,10));
 isempty
---------
 f
(1 row)
```

## 给范围类型创建索引

```sql
mydb=# create index idx_ip_adress_range on ip_address using gist(ip_range);
ERROR:  relation "ip_address" does not exist
```

> 书上解释太少，命令还有些问题

# json/jsonb 类型



## json 类型简介

```sql
-- json 简单例子
mydb=# select '{"a":1,"b":2}'::json;
     json
---------------
 {"a":1,"b":2}
(1 row)

mydb=# create table test_json1(id serial primary key,name json);
CREATE TABLE
mydb=# insert into test_json1(name) values('{"col1":1,"col2":"francs","col3":"male"}');
INSERT 0 1
mydb=# insert into test_json1(name) values('{"col1":1,"col2":"frances","col3":"male"}');
INSERT 0 1
mydb=# select * from test_json1;
 id |                   name
----+-------------------------------------------
  1 | {"col1":1,"col2":"francs","col3":"male"}
  2 | {"col1":1,"col2":"frances","col3":"male"}
(2 rows)
```

## 查询 json 数据

```sql
-- 通过 "->" 操作符可以查询 json 数据的键值
mydb=# select name -> 'col2' from test_json1 where id=1;
 ?column?
----------
 "francs"
(1 row)
-- 文本格式返回 json 字段键值可以使用 "->>" 操作符
mydb=# select name ->> 'col2' from test_json1 where id=1;
 ?column?
----------
 francs
(1 row)
```

## jsonb 与 json 差异

| json                           | jsonb                             |
| ------------------------------ | --------------------------------- |
| 存储格式为文本(存储快，使用慢) | 存储格式为二进制(存储慢，使用快)  |
| json输出键的顺序和输入完全一样 | 输出的键的顺序和输入不一样        |
| 不会去掉空格                   | 去掉输入数据中键值的空格          |
| 不会删除重复的键               | 会删除重复的键,**仅保留最后一个** |

```sql
-- 输出键值顺序差异
mydb=# select '{"bar":"baz","balance":7.77,"active":false}'::jsonb;
                      jsonb
--------------------------------------------------
 {"bar": "baz", "active": false, "balance": 7.77}
(1 row)

mydb=# select '{"bar":"baz","balance":7.77,"active":false}'::json;
                    json
---------------------------------------------
 {"bar":"baz","balance":7.77,"active":false}
(1 row)

-- 输入空格差异
mydb=# select '{"id":1,  "name":"francs"}'::jsonb;
            jsonb
-----------------------------
 {"id": 1, "name": "francs"}
(1 row)

mydb=# select '{"id":1,  "name":"francs"}'::json;
            json
----------------------------
 {"id":1,  "name":"francs"}
(1 row)
-- 重复键的差异
mydb=# select '{"id":1,"name":"francs","remark":"a good guy!","name":"test"}'::jsonb;
                       jsonb
----------------------------------------------------
 {"id": 1, "name": "test", "remark": "a good guy!"}
(1 row)

mydb=# select '{"id":1,"name":"francs","remark":"a good guy!","name":"test"}'::json;
                             json
---------------------------------------------------------------
 {"id":1,"name":"francs","remark":"a good guy!","name":"test"}
(1 row)
```

> 在大多数应用场景下建议使用 jsonb, 除非有特殊需求，比如对 json 键顺序

## jsonb 与 json 操作符

```sql
-- 以文本格式返回 json 类型 "->>"
mydb=# select name ->> 'col2' from test_json1 where id =1;
 ?column?
----------
 francs
(1 row)
-- 字符串是否作为顶层键值
mydb=# select '{"a":1,"b":2}'::jsonb ? 'a';
 ?column?
----------
 t
(1 row)
-- 删除 json 数据的键/值
mydb=# select '{"a":1,"b":"faasd"}'::jsonb - 'a';
    ?column?
----------------
 {"b": "faasd"}
(1 row)
```

## Jsonb 与 json 函数



```sql
-- 扩展最外层的 json 对象成为一组键/值结果集
mydb=# select * from json_each('{"a":"foo","b":"bar"}');
 key | value
-----+-------
 a   | "foo"
 b   | "bar"
(2 rows)
-- 文本方式返回结果
mydb=# select * from json_each_text('{"a":"foo","b":"bar"}');
 key | value
-----+-------
 a   | foo
 b   | bar
(2 rows)
-- row_to_json 函数按行输出
mydb=# select * from test_copy where id =1;
 id | name
----+------
  1 | a
  1 | a
(2 rows)

mydb=# select row_to_json(test_copy) from test_copy where id =1;
     row_to_json
---------------------
 {"id":1,"name":"a"}
 {"id":1,"name":"a"}
(2 rows)
-- 返回最外层的 json 对象中的键的集合
mydb=# select * from json_object_keys('{"a":"foo","b":"bar"}');
 json_object_keys
------------------
 a
 b
(2 rows)
```

## jsonb 键/值的追加、删除、更新

```sql
-- || 追加
mydb=# select '{"name":"francs","age":"31"}'::jsonb || '{"sex":"male"}'::jsonb;
                    ?column?
------------------------------------------------
 {"age": "31", "sex": "male", "name": "francs"}
(1 row)
-- 删除  通过“-” 通过“#-” 常用于有嵌套的 json 数据删除
mydb=# select '{"name":"james","email":"james@localhost"}'::jsonb - 'email';
     ?column?
-------------------
 {"name": "james"}
(1 row)
-- 这里没弄明白 #- 后面那块内容(应该参数1：删除key，参数2 删掉嵌套的位置)
mydb=# select '{"name":"James","contact":{"phone":"0234123123","fax":"1231231"}}'::jsonb #- '{contact,fax}'::text[];
                       ?column?
-------------------------------------------------------
 {"name": "James", "contact": {"phone": "0234123123"}}
(1 row)
-- 删除嵌套 aliases 中位置 1的 键/值(⚠️这里因为是数组，所以是数组索引)
mydb=# select '{"name":"asd","aliases":["ads","sdasd","asdasd"]}'::jsonb #- '{aliases,0}'::text[];
                    ?column?
-------------------------------------------------
 {"name": "asd", "aliases": ["sdasd", "asdasd"]}
(1 row)

-- 更新 “||”操作符，即可连接 json 也可以覆盖重复的键值
mydb=# select '{"name":"francs","age":"31"}'::jsonb || '{"age":"14"}'::jsonb;
            ?column?
---------------------------------
 {"age": "14", "name": "francs"}
(1 row)

mydb=# select '{"name":"francs","age":"31"}'::jsonb || '{"age":"14"}';
            ?column?
---------------------------------
 {"age": "14", "name": "francs"}
(1 row)
-- 更新2 jsonb_set(target jsonb,path text[],new_value jsonb[,create_missing],boolean);
-- targer 源 jsonb 数据 path 路径，new_value 指更新后的键值 
-- create_missing  true 如果键不存在则添加 false 如果键不存在则不添加
mydb=# select jsonb_set('{"name":"francs","age":"31"}'::jsonb,'{age}','"123"'::jsonb,false);
            jsonb_set
----------------------------------
 {"age": "123", "name": "francs"}
(1 row)

mydb=# select jsonb_set('{"name":"francs","age":"31"}'::jsonb,'{addAge}','"123"'::jsonb,true);
                    jsonb_set
--------------------------------------------------
 {"age": "31", "name": "francs", "addAge": "123"}
(1 row)
```



# 数据类型转换

- 格式化函数
- CAST函数
- `::`操作符

## 通过格式化函数进行转化

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gi2wz9raznj30vu0di7fi.jpg)

```sql
-- 简单尝试
mydb=# select to_date('05 Dec 2000','DD Mon YYYY');
  to_date
------------
 2000-12-05
(1 row)

mydb=# select to_number('123,131.4-','99F999D9S');
 to_number
-----------
     12131
(1 row)
```

> 数据格式参考相关链接

## 通过CAST函数进行转换

```sql
-- varchar 字符转换为 text
mydb=# select cast(varchar '123' as text);
 text
------
 123
(1 row)
-- varchar 字符类型转换 int4
 mydb=# select cast(varchar '123' as int4);
 int4
------
  123
(1 row)
```



## 通过：：操作符进行转换

```sql
-- 转换成 int4 或 numeric 类型
mydb=# select 1::int4,3/2::numeric;
 int4 |      ?column?
------+--------------------
    1 | 1.5000000000000000
(1 row)
```

```sql
-- 查询表 test_json1 在系统表 pg_class 的 oid 和 relname
mydb=# select oid,relname from pg_class where relname='test_json1';
  oid  |  relname
-------+------------
 16504 | test_json1
(1 row)
-- 根据 oid 在系统表 pg_attribute 中 attrelid 找到表的字段
mydb=# select attname from pg_attribute where attrelid='16504' and attnum > 0;
 attname
---------
 id
 name
(2 rows)
-- :: 类型转换替换上面两条语句
mydb=# select attname from pg_attribute where attrelid = 'test_json1'::regclass and attnum > 0;
 attname
---------
 id
 name
(2 rows)
```

> 这里不明白 regclass 怎么就把对应的id转换了？

> 解释:
>
> - pg_class 系统表存储 PostgreSQL 对象信息，比如表、索引、序列、试图等，OID 是隐藏字段，唯一标识 pg_class 中的一行，可以理解成 pg_class 系统表的对象ID；
> - pg_attribute 系统表存储表的字段信息，数据库表的每一个字段在这个视图中都有相应的一条记录
> - pg_attribute.attrelid 是指字段所属表 OID

# 相关链接

- [postgresql----JSON和JSONB类型](https://blog.csdn.net/u012129558/article/details/81453640)
- [PostgreSQL中的时间戳格式转化常识](https://my.oschina.net/u/3760785/blog/1930575)