# PostgreSQL 的 NoSQL 特性

PostgreSQL 不只是一个关系型数据库，同时支持非关系特性

本节内容：

- json 和 jsonb 数据类型
	- 为 jsonb 类型创建索引
	- json 和 jsonb 读写性能测试
	- 全文检索对 json 和 jsonb 数据类型的支持

# 1 为 jsonb 类型创建索引

jsonb 数据类型**支持 GIN 所有**，为了便于说明，例如一个 json 字符内容如下所示，并且以 jsonb 格式存储：

```json
{
    "id":1,
    "user_id":12312,
    "user_name":"1_frances",
    "create_time":"2011-12-12 12:12:12.21313+08"
}
```

加入存储以上 jsonb 数据的字段名 为 user_info，表 tbl_user_jsonb,在 user_info 字段上创建 GIN 索引

```sql
create table tl_user_jsonb (id serial primary key,user_info jsonb);
CREATE TABLE

create index idx_gin ON tbl_user_jsonb USING gin(user_info);
```

# 2 json、jsonb 读写性能测试



## 1 创建 json、jsonb 测试表

下面通过一个简单的例子测试 json、jsonb 的读写性能差异，计划创建一下三种表：

- `user_ini`:基础数据表，并插入 200万测试数据
- `tbl_user_json`:json 数据类型表，200万
- `tbl_user_jsonb`:jsonb 数据类型表，200万

```sql
mydb=# create table user_ini(
    id int4,
    user_id int8,
    user_name character varying(64),
    create_time timestamp(6) with time) zone default clock_timestamp());
CREATE TABLE
mydb=# create table tbl_user_json(id serial,user_info json);
CREATE TABLE
mydb=# create table tbl_user_jsonb(id serial,user_info jsonb);
CREATE TABLE
-- user_ini 插入 200万条数据
mydb=# insert into user_ini(id,user_id,user_name)
mydb-# select r,round(random()*2000000),r || '_francs'
mydb-# from generate_series(1,2000000) as r;
INSERT 0 2000000 
```

## 2 json、jsonb 表写性能测试

根据 user_ini 数据通过 `row_to_json` 函数向表 user_ini_json 插入 200 万

```sql
mydb=# \timing
Timing is on.
-- json 插入 200万
mydb=# insert into tbl_user_json(user_info) select row_to_json(user_ini)
mydb-# from user_ini;
INSERT 0 2000000
Time: 12614.474 ms (00:12.614)

-- jsonb 插入 200 万
mydb=# insert into tbl_user_jsonb(user_info)
select row_to_json(user_ini)::jsonb from user_ini;
INSERT 0 2000000
Time: 20197.870 ms (00:20.198)
-- 比较存储大小
mydb=# \dt+ tbl_user_json
                        List of relations
 Schema |     Name      | Type  |  Owner   |  Size  | Description
--------+---------------+-------+----------+--------+-------------
 public | tbl_user_json | table | postgres | 281 MB |
(1 row)

mydb=# \dt+ tbl_user_jsonb
                         List of relations
 Schema |      Name      | Type  |  Owner   |  Size  | Description
--------+----------------+-------+----------+--------+-------------
 public | tbl_user_jsonb | table | postgres | 333 MB |
(1 row)
```

## 3 json、jsonb 表读性能测试

根据 user_info 字段的 user_namer 键的值查询

```sql
mydb=# explain analyze select * from tbl_user_jsonb where user_info->> 'user_name' = '1_francs';
                                                            QUERY PLAN
------------------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..57053.18 rows=10000 width=143) (actual time=0.389..582.010 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Parallel Seq Scan on tbl_user_jsonb  (cost=0.00..55053.18 rows=4167 width=143) (actual time=367.695..560.025 rows=0 loops=3)
         Filter: ((user_info ->> 'user_name'::text) = '1_francs'::text)
         Rows Removed by Filter: 666666
 Planning Time: 0.187 ms
 Execution Time: 582.042 ms
(8 rows)
-- tbl_user_json 是 3千秒，这里忘记赋值了
-- 基于 user_info 字段 user_name 键值创建 btree 索引
mydb=# create index idx_jsonb on tbl_user_jsonb Using btree ((user_info ->> 'user_name'));
CREATE INDEX
Time: 8285.675 ms (00:08.286)
mydb=# create index idx_json on tbl_user_json Using btree ((user_info ->> 'user_name'));
CREATE INDEX
Time: 9959.782 ms (00:09.960)
-- 再次执行计划 jsonb
mydb=# explain analyze select * from tbl_user_jsonb where user_info->> 'user_name' = '1_francs';
                                                         QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on tbl_user_jsonb  (cost=233.93..23868.23 rows=10000 width=143) (actual time=0.205..0.206 rows=1 loops=1)
   Recheck Cond: ((user_info ->> 'user_name'::text) = '1_francs'::text)
   Heap Blocks: exact=1
   ->  Bitmap Index Scan on idx_jsonb  (cost=0.00..231.43 rows=10000 width=0) (actual time=0.192..0.192 rows=1 loops=1)
         Index Cond: ((user_info ->> 'user_name'::text) = '1_francs'::text)
 Planning Time: 0.355 ms
 Execution Time: 0.231 ms
(7 rows)
-- json 的执行计划
                                                       QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on tbl_user_json  (cost=233.93..22474.05 rows=10000 width=113) (actual time=0.028..0.029 rows=1 loops=1)
   Recheck Cond: ((user_info ->> 'user_name'::text) = '1_francs'::text)
   Heap Blocks: exact=1
   ->  Bitmap Index Scan on idx_json  (cost=0.00..231.43 rows=10000 width=0) (actual time=0.023..0.023 rows=1 loops=1)
         Index Cond: ((user_info ->> 'user_name'::text) = '1_francs'::text)
 Planning Time: 0.071 ms
 Execution Time: 0.050 ms
(7 rows)

```

为了更好对比 tbl_user_json、tbl_user_jsonb 表基于键值查询的效率，根据 user_info 字段的id 键进行扫描以对比性能

```sql
mydb=# create index idx_gin_user_info_id ON tbl_user_json USING btree (((user_info ->> 'id')::integer));
CREATE INDEX
Time: 3927.422 ms (00:03.927)
mydb=# create index idx_gin_user_infob_id ON tbl_user_jsonb USING btree (((user_info ->> 'id')::integer));
CREATE INDEX
Time: 2508.140 ms (00:02.508)
```

创建索引后查询表：

```sql
mydb=# explain analyze select id,user_info->'id',user_info->'user_name'
mydb-# from tbl_user_json
mydb-# where (user_info->>'id')::int4>1 and (user_info->>'id')::int4<10000;
Time: 42.778 ms
                                                              QUERY PLAN
--------------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on tbl_user_json  (cost=214.93..22655.05 rows=10000 width=68) (actual time=0.733..30.340 rows=9998 loops=1)
   Recheck Cond: ((((user_info ->> 'id'::text))::integer > 1) AND (((user_info ->> 'id'::text))::integer < 10000))
   Heap Blocks: exact=173
   ->  Bitmap Index Scan on idx_gin_user_info_id  (cost=0.00..212.43 rows=10000 width=0) (actual time=0.676..0.677 rows=9998 loops=1)
         Index Cond: ((((user_info ->> 'id'::text))::integer > 1) AND (((user_info ->> 'id'::text))::integer < 10000))
 Planning Time: 1.061 ms
 Execution Time: 31.695 ms
(7 rows)

-- 查询 jonb

mydb=# explain analyze select id,user_info->'id',user_info->'user_name'
from tbl_user_jsonb
where (user_info->>'id')::int4>1 and (user_info->>'id')::int4<10000;
                                                              QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on tbl_user_jsonb  (cost=214.93..24049.23 rows=10000 width=68) (actual time=0.663..8.749 rows=9998 loops=1)
   Recheck Cond: ((((user_info ->> 'id'::text))::integer > 1) AND (((user_info ->> 'id'::text))::integer < 10000))
   Heap Blocks: exact=212
   ->  Bitmap Index Scan on idx_gin_user_infob_id  (cost=0.00..212.43 rows=10000 width=0) (actual time=0.625..0.626 rows=9998 loops=1)
         Index Cond: ((((user_info ->> 'id'::text))::integer > 1) AND (((user_info ->> 'id'::text))::integer < 10000))
 Planning Time: 0.261 ms
 Execution Time: 9.490 ms
(7 rows)

```

- 验证 **json写入比jsonb快，但检索时比jsonb慢**

# 3 全文减速对 json 和 jsonb 数据类型的支持