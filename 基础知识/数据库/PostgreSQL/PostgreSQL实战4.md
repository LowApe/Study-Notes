# SQL 高级特性

关键词:

- with 查询
- 批量插入
- returning 返回修改的数据
- upsert
- 数据抽样
- **聚合函数**
- 窗口函数

# WITH 查询

> WITH 查询在复杂查询中定义的一个辅助语句,也就是**临时表**，**通常用于复杂查询或递归查询**

## 复杂查询

```sql
with temp as (select generate_series(1,3)) 
select * from temp;
 generate_series
-----------------
               1
               2
               3
(3 rows)
```

## 递归查询

```sql
with recursive t(x) as(
    select 1
    union 
    select x+1 
    from t 
    where x<5
) 
select sum(x) from t;
 sum
-----
  15
(1 row)
```

上面的例子x从1开始，union加1后的值，循环直到 x 小于5结束，之后计算 x 值的总和。

**地址的例子**,

> 使用 with recursive 递归方式根据一个id查询该id下所有关联

```sql
-- 初始化数据
create table test_area(id int4,name varchar(32),parent_id int4);
insert into test_area values(1,'中国',0);
insert into test_area values(2,'辽宁',1);
insert into test_area values(3,'山东',1);
insert into test_area values(4,'沈阳',2);
insert into test_area values(5,'大连',2);
insert into test_area values(6,'济南',3);
insert into test_area values(7,'和平区',4);
insert into test_area values(8,'南开区',4);

-- 查看
 select * from test_area;
 id |  name  | parent_id
----+--------+-----------
  1 | 中国   |         0
  2 | 辽宁   |         1
  3 | 山东   |         1
  4 | 沈阳   |         2
  5 | 大连   |         2
  6 | 济南   |         3
  7 | 和平区 |         4
  8 | 南开区 |         4
  
 -- with 递归查询
 with recursive detail as(
 	select * from test_area where id =7
     union all
     select test_area.* from test_area,detail 
     where test_area.id = detail.parent_id
 )
 
 select * from detail order by id;
 
 
  id |  name  | parent_id
----+--------+-----------
  1 | 中国   |         0
  2 | 辽宁   |         1
  4 | 沈阳   |         2
  7 | 和平区 |         4
(4 rows)

-- 平铺展示 用到了 string_agg 函数
mydb=# with recursive detail as(
select * from test_area where id =7 union all
select test_area.* from test_area,detail
where test_area.id = detail.parent_id)
select string_agg(name,'') from (select name from detail order by id) n;
     string_agg
--------------------
 中国辽宁沈阳和平区
(1 row)

```

> :sos:这里with的示例还不太清楚，后面有机会过来在总结一下​

# 批量查询

## 批量插入

### 方式1 insert into ... select ...

```sql
mydb=# create table tbl_batch2(id int4,info text);
CREATE TABLE
mydb=# insert into tbl_batch2(id,info) select generate_series(1,5),'batch2';
INSERT 0 5
mydb=# select * from tbl_batch2;
```



### 方式2 insert into values(),(),...()

```sql
mydb=# create table tbl_batch3(id int4,info text);
CREATE TABLE
mydb=# insert into tbl_batch3 values (1,'a'),(2,'b');
INSERT 0 2
mydb=# select * from tbl_batch3;
 id | info
----+------
  1 | a
  2 | b
(2 rows)

```



### 方式3 copy 或 \copy 元命令

> 导入、导出效率相比insert 命令插入销量更高，**通常大数据量的文件导入一般在数据库服务端主机通过 COPY 命令导入**



# Returning 返回修改的数据

> returning 特性可以返回 DML 修改的数据
>
> - insert 语句后接 returning 属性返回插入的数据
> - update 后接 returning 属性返回更新后的新值
> - delete 语句后接 returning 属性返回删除的数据
>
> **这个特性的优点在于不需要额外的 SQL 获取这些值，能够方便应用开发**



```sql
-- insert returning * *:表示返回所有的列，也可以返回指定的列
mydb=# insert into
tbl_batch3 values(1,'c') returning *;
 id | info
----+------
  1 | c
(1 row)
INSERT 0 1
-- 不加 returning
mydb=# insert into
tbl_batch3 values(4,'d') ;
INSERT 0 1

-- update
mydb=# select * from tbl_batch3;
 id | info
----+------
  1 | a
  2 | b
  1 | c
  4 | d
(4 rows)


mydb=# update tbl_batch3 set id = 3 where info = 'c' returning *;
 id | info
----+------
  3 | c
(1 row)
UPDATE 1


```



# Upsert

> Upset 特性是指 **insert ... on  confict update**,用来解决在数据插入过程中数据冲突的情况，比如在事务中批量插入日志数据，如果其中有一条数据违反表上的约束，则整个插入事务将会回滚，postgreSQL 的 Upsert 特性能解决这一问题



## upsert 场景演示

```sql
mydb=# create table user_logs(user_name text primary key,login_cnt int4,last_login_time timestamp(0) without time zone);
CREATE TABLE

-- 插入一条数据
insert into user_logs values('frances',1);
-- 批量插入重复 text 注解会报错
mydb=# insert into user_logs values('matiler',1),('frances',1);
ERROR:  duplicate key value violates unique constraint "user_logs_pkey"
DETAIL:  Key (user_name)=(matiler) already exists.
```

> 批量插入两条数据，其中 matiler 这条数据不违反逐渐冲突，upsert 可以处理冲突的数据

```sql
mydb=# insert into user_logs(user_name,login_cnt)
values ('matiler',1),('frances',1)
on conflict (user_name)
do update set
login_cnt=user_logs.login_cnt+EXCLUDED.login_cnt,last_login_time=now();
INSERT 0 2
mydb=# select * from user_logs
;
 user_name | login_cnt |   last_login_time
-----------+-----------+---------------------
 matiler   |         1 |
 frances   |         2 | 2020-09-28 18:16:20
(2 rows)
```

> insert 语句插入两条数据，并设置规则：当数据冲突时将登陆次数字段 login_cnt 值 +1 ，同时更新最近登陆 last_login_time, **on conflict 定义冲突字段 user_name，do update set  冲突动作后更新这个冲突，**

```sql
-- 也可以定义冲突后什么都不做 do nothing
mydb=# insert into user_logs(user_name,login_cnt)
values ('abc',1),('frances',1)
on conflict (user_name)
do nothing

```



### Upsert 语法

```sql
INSERT INTO table_name [ AS alias ] [ ( column_name [, ...] ) ]
    [ OVERRIDING { SYSTEM | USER } VALUE ]
    { DEFAULT VALUES | VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
    [ ON CONFLICT [ conflict_target ] conflict_action ]
    [ RETURNING * | output_expression [ [ AS ] output_name ] [, ...] ]

where conflict_target can be one of:

    ( { index_column_name | ( index_expression ) } [ COLLATE collation ] [ opclass ] [, ...] ) [ WHERE index_predicate ]
    ON CONSTRAINT constraint_name

and conflict_action is one of:

    DO NOTHING
    DO UPDATE SET { column_name = { expression | DEFAULT } |
                    ( column_name [, ...] ) = [ ROW ] ( { expression | DEFAULT } [, ...] ) |
                    ( column_name [, ...] ) = ( sub-SELECT )
                  } [, ...]
              [ WHERE condition ]
```



# 数据抽样

> 当表数据量比较大时，随机查询表中一定数量记录的操作很常见。9.5 版本以后 PostgreSQL 支持 TABLESAMPLE 数据抽样，

```sql
select ...
from table_name
tablesample sampling_method (argument[,...])[repeatable(seed)]
```

- sampling_method 指抽样方法，主要有两种: SYSTEM 和 BERNOULLI 

## SYSTEM 抽样方式

> 表示随机抽取表上**数据块**上的数据，

```sql
-- 创建表
mydb=# create table  table_sample(id int4,message text,create_time timestamp(6) without time zone default clock_timestamp());
CREATE TABLE

-- 插入 15万数据
mydb=# insert into table_sample(id,message)
select n,md5(random()::text) from generate_series(1,150000) n;
INSERT 0 150000

-- 获取一条
mydb=# select * from table_sample limit 1;
 id |             message              |        create_time
----+----------------------------------+----------------------------
  1 | 485aee6cc1670e2e2b287c41205a68e5 | 2020-09-28 19:42:51.809087
(1 row)

-- 抽样因子设置为 0.01,意味着返回 150000 x 0.01% = 15
mydb=# explain analyse select * from table_sample tablesample system(0.01);
                                                 QUERY PLAN
-------------------------------------------------------------------------------------------------------------
 Sample Scan on table_sample  (cost=0.00..4.15 rows=15 width=45) (actual time=0.032..0.138 rows=107 loops=1)
   Sampling: system ('0.01'::real)
 Planning Time: 0.380 ms
 Execution Time: 0.278 ms
(4 rows)

-- 优化器预计访问 15 条，实际返回 107条，因为表占用的数据块
-- 150000/1402 = 107
mydb=# select relname,relpages from pg_class where relname='table_sample';
   relname    | relpages
--------------+----------
 table_sample |     1402
(1 row)

```

> ⚠️注意：
>
> - explain analyze 命令表示实际执行这条 SQL，同时显示 SQL 执行计划和执行时间
> - planning time 表示 SQL语句解析生成执行计划的时间
> - execution time 表示 SQL 的实际执行时间

## bernoulli 抽样方式

bernoulli 抽样方式随机抽取表的**数据行**，并**返回指定百分比数据**。

```sql
mydb=# explain analyze select * from table_sample tablesample bernoulli(0.01);
                                                  QUERY PLAN
---------------------------------------------------------------------------------------------------------------
 Sample Scan on table_sample  (cost=0.00..1402.15 rows=15 width=45) (actual time=0.193..2.931 rows=16 loops=1)
   Sampling: bernoulli ('0.01'::real)
 Planning Time: 0.045 ms
 Execution Time: 2.943 ms
(4 rows)
```

> 抽取基于数据行级别，返回的数据在接近百分比



# 聚合函数

常用的聚合函数

- avg()
- sum()
- min()
- max()
- count()

## string_agg函数

> string_agg 能将结果集**某个字段的所有行连接成字符串，并用指定 delimiter 分隔符分隔，expression表示要处理的字符类型数据(字段)**

```sql
-- 语法
string_agg(expression,delimiter);
-- 测试数据
mydb=# create table city (country character varying(64),city character varying(64));
CREATE TABLE
insert into city values('中国','台北'),('中国','香港'),('中国','上海'),('日本','东京'),('日本','大阪');
INSERT 0 5
mydb=# select * from city;
 country | city
---------+------
 中国    | 台北
 中国    | 香港
 中国    | 上海
 日本    | 东京
 日本    | 大阪
(5 rows)
-- 将所有结果行拼接
mydb=# select string_agg(city,',') from city;
        string_agg
--------------------------
 台北,香港,上海,东京,大阪
(1 row)
-- 
mydb=# select country,string_agg(city,',') from city group by country;
 country |   string_agg
---------+----------------
 日本    | 东京,大阪
 中国    | 台北,香港,上海
(2 rows)
```

## array_agg 函数

> array_agg 函数返回的类型为数组，数组数据类型同输入参数类型一致

```sql
-- 语法
array_agg(expression) -- 输入参数为任何非数组类型

-- 查询
mydb=# select country,array_agg(city) from city group by country;
 country |    array_agg
---------+------------------
 日本    | {东京,大阪}
 中国    | {台北,香港,上海}
(2 rows)

-- 语法
array_agg(expression) -- 输入参数为任何数组类型
mydb=# create table test_array3(id int4[]);
CREATE TABLE
                                                                   ^
mydb=# insert into test_array3 values(array[1,2,3]),(array[4,5,6]);
INSERT 0 2

mydb=# select * from test_array3;
   id
---------
 {1,2,3}
 {4,5,6}
(2 rows)
mydb=# select array_agg(id) from test_array3;
     array_agg
-------------------
 {{1,2,3},{4,5,6}}
(1 row)

-- 指定 array_to_string 函数，指定分隔符

```

# 窗口函数

窗口函数是**基于结果集计算，将计算出的结果合并到输出的结果集上,并返回多行**

## 窗口函数语法

> PostgreSQL 提供内置的窗口函数，例如 row_num()、rank()、log() 等，除了内置的窗口函数外，聚合函数、自定义函数后接 OVER 属性也可作为窗口函数。



```sql
-- 窗口函数语法
function_name([expression][,expression ...])[FILTER(where filter_clause)]
over (window_definition)

-- window_definition
[existing_window]
[ partition by expression [,...]]
[ORDER BY expression [ASC | DESC | USING operator]]
```

- OVER 表示窗口函数的关键字
- PARTITION BY 属性对查询返回的结果集进行分组，之后窗口函数处理分组的数据
- ORDER BY 设定结果集的分组数据排序

### avg() OVER()

```sql
mydb=# create table score(
    id serial primary key,
    subject character varying(32),
	stu_name character varying(32),
    score numeric(3,0));
CREATE TABLE
mydb=# insert into  score(subject,stu_name,score) values
('Chinese','fransce',70),
('Chinese','matiler',70),
('Chinese','tutu',80),
('English','matiler',75),
('English','fransces',90),
('English','tutu',60),
('Math','fransce',80),
('Math','matiler',99),
('Math','tutu',65);
INSERT 0 9
-- 查询每名学生学习成绩并且显示课程的平均分
mydb=# select s.subject,s.stu_name,s.score,tmp.avgscore
from score s
left join (select subject,avg(score) avgscore from score group by subject) tmp on s.subject = tmp.subject;
 subject | stu_name | score |      avgscore
---------+----------+-------+---------------------
 Chinese | fransce  |    70 | 73.3333333333333333
 Chinese | matiler  |    70 | 73.3333333333333333
 Chinese | tutu     |    80 | 73.3333333333333333
 English | matiler  |    75 | 75.0000000000000000
 English | fransces |    90 | 75.0000000000000000
 English | tutu     |    60 | 75.0000000000000000
 Math    | fransce  |    80 | 81.3333333333333333
 Math    | matiler  |    99 | 81.3333333333333333
 Math    | tutu     |    65 | 81.3333333333333333
(9 rows)

-- 使用窗口函数很容实现以上需求
mydb=# select *,avg(score) over(partition by subject) from score;
 id | subject | stu_name | score |         avg
----+---------+----------+-------+---------------------
  1 | Chinese | fransce  |    70 | 73.3333333333333333
  2 | Chinese | matiler  |    70 | 73.3333333333333333
  3 | Chinese | tutu     |    80 | 73.3333333333333333
  4 | English | matiler  |    75 | 75.0000000000000000
  5 | English | fransces |    90 | 75.0000000000000000
  6 | English | tutu     |    60 | 75.0000000000000000
  7 | Math    | fransce  |    80 | 81.3333333333333333
  8 | Math    | matiler  |    99 | 81.3333333333333333
  9 | Math    | tutu     |    65 | 81.3333333333333333
(9 rows)
```

### row_number()

> Row_number() 窗口函数对**结果集分组后的数据标注行号**，从 1 开始，如下所示:

```sql
mydb=# select *,avg(score) over(partition by subject) from score;
 id | subject | stu_name | score |         avg
----+---------+----------+-------+---------------------
  1 | Chinese | fransce  |    70 | 73.3333333333333333
  2 | Chinese | matiler  |    70 | 73.3333333333333333
  3 | Chinese | tutu     |    80 | 73.3333333333333333
  4 | English | matiler  |    75 | 75.0000000000000000
  5 | English | fransces |    90 | 75.0000000000000000
  6 | English | tutu     |    60 | 75.0000000000000000
  7 | Math    | fransce  |    80 | 81.3333333333333333
  8 | Math    | matiler  |    99 | 81.3333333333333333
  9 | Math    | tutu     |    65 | 81.3333333333333333
(9 rows)


```

> 如果不指定 partition 属性， row_number() 窗口函数显示表所有记录的行号

### rank()

> **当组内某行字段值相同时，行号重复并且行号产生间隙**,如下所示：

```sql
mydb=# select rank() OVER(PARTITION BY subject order by score),* from score;
 rank | id | subject | stu_name | score
------+----+---------+----------+-------
    1 |  2 | Chinese | matiler  |    70
    1 |  1 | Chinese | fransce  |    70
    3 |  3 | Chinese | tutu     |    80
    1 |  6 | English | tutu     |    60
    2 |  4 | English | matiler  |    75
    3 |  5 | English | fransces |    90
    1 |  9 | Math    | tutu     |    65
    2 |  7 | Math    | fransce  |    80
    3 |  8 | Math    | matiler  |    99
(9 rows)
```

### dense_rank()

> 当组内某行字段值相同时，虽然行号重复，但行号不产生间隙

### lag()

> 可以获取行偏移 offset 那行某个字段的数据
>
> :sos:不知道这个偏移有啥用

```sql
lag(value anyelement [,offset integer [,default anyelement]])
```

- value  指定要返回记录的字段
- offset 行偏移量,默认值 1，可以是正整数或负整数，正整数表示取结果集中向上偏移的记录，负整数表示取结果集中向下偏移的记录
- default 是指如果不存在 offset 偏移的行时用默认值填充，default 值默认为null 

```sql
-- id 偏移 1位
mydb=# select lag(id,1) over(),* from score;
 lag | id | subject | stu_name | score
-----+----+---------+----------+-------
     |  1 | Chinese | fransce  |    70
   1 |  2 | Chinese | matiler  |    70
   2 |  3 | Chinese | tutu     |    80
   3 |  4 | English | matiler  |    75
   4 |  5 | English | fransces |    90
   5 |  6 | English | tutu     |    60
   6 |  7 | Math    | fransce  |    80
   7 |  8 | Math    | matiler  |    99
   8 |  9 | Math    | tutu     |    65
(9 rows)
-- 偏移两位 并且设置默认值

mydb=# select lag(id,2,999) over(),* from score;
 lag | id | subject | stu_name | score
-----+----+---------+----------+-------
 999 |  1 | Chinese | fransce  |    70
 999 |  2 | Chinese | matiler  |    70
   1 |  3 | Chinese | tutu     |    80
   2 |  4 | English | matiler  |    75
   3 |  5 | English | fransces |    90
   4 |  6 | English | tutu     |    60
   5 |  7 | Math    | fransce  |    80
   6 |  8 | Math    | matiler  |    99
   7 |  9 | Math    | tutu     |    65
(9 rows)
```

### first_value()

> first_value窗口函数用来取**结果集每一个分组的第一行数据的字段值**

```sql
mydb=# select first_value(score) over(partition by subject),* from score;
 first_value | id | subject | stu_name | score
-------------+----+---------+----------+-------
          70 |  1 | Chinese | fransce  |    70
          70 |  2 | Chinese | matiler  |    70
          70 |  3 | Chinese | tutu     |    80
          75 |  4 | English | matiler  |    75
          75 |  5 | English | fransces |    90
          75 |  6 | English | tutu     |    60
          80 |  7 | Math    | fransce  |    80
          80 |  8 | Math    | matiler  |    99
          80 |  9 | Math    | tutu     |    65
(9 rows)

-- 通过排序获取每组的最高值
mydb=# select first_value(score) over(partition by subject order by score desc),* from score;
 first_value | id | subject | stu_name | score
-------------+----+---------+----------+-------
          80 |  3 | Chinese | tutu     |    80
          80 |  1 | Chinese | fransce  |    70
          80 |  2 | Chinese | matiler  |    70
          90 |  5 | English | fransces |    90
          90 |  4 | English | matiler  |    75
          90 |  6 | English | tutu     |    60
          99 |  8 | Math    | matiler  |    99
          99 |  7 | Math    | fransce  |    80
          99 |  9 | Math    | tutu     |    65
(9 rows)

```

### last_value()

> 取结果集每一个分组的最后一行数据的字段值



### nth_value()

> 取结果集每个分组的指定行数据的字段值

```sql
-- 语法
nth_value(value any,nth integer)
-- 根据分数，查看第3个
mydb=# select nth_value(score,3) over(partition by subject),* from score;
 nth_value | id | subject | stu_name | score
-----------+----+---------+----------+-------
        80 |  1 | Chinese | fransce  |    70
        80 |  2 | Chinese | matiler  |    70
        80 |  3 | Chinese | tutu     |    80
        60 |  4 | English | matiler  |    75
        60 |  5 | English | fransces |    90
        60 |  6 | English | tutu     |    60
        65 |  7 | Math    | fransce  |    80
        65 |  8 | Math    | matiler  |    99
        65 |  9 | Math    | tutu     |    65
(9 rows)
```

### 窗口函数别名的使用

> 需要多次使用窗口函数

```sql
select avg(score),over(r),sum(score) over(r),* from score 
window r as(partition by subject);
         avg         | sum | id | subject | stu_name | score
---------------------+-----+----+---------+----------+-------
 73.3333333333333333 | 220 |  1 | Chinese | fransce  |    70
 73.3333333333333333 | 220 |  2 | Chinese | matiler  |    70
 73.3333333333333333 | 220 |  3 | Chinese | tutu     |    80
 75.0000000000000000 | 225 |  4 | English | matiler  |    75
 75.0000000000000000 | 225 |  5 | English | fransces |    90
 75.0000000000000000 | 225 |  6 | English | tutu     |    60
 81.3333333333333333 | 244 |  7 | Math    | fransce  |    80
 81.3333333333333333 | 244 |  8 | Math    | matiler  |    99
 81.3333333333333333 | 244 |  9 | Math    | tutu     |    65
(9 rows)
```



# 相关问题



# 相关链接