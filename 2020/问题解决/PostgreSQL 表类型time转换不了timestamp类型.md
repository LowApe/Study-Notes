

# PostgreSQL 表类型 time 转换不了 timeStamp类型

```sql

ALTER TABLE "public"."xxxx" ALTER COLUMN "xxxx" TYPE TIMESTAMP USING  xxxx::timestamp without time zone

Query 1 ERROR: ERROR:  cannot cast type time without time zone to timestamp without time zone
LINE 1: ...erating_time" TYPE TIMESTAMP USING  operating_time::timestam...
                                                             ^
```

> 死活转不了，即使根据提示，还有百度都尝试了，检查了好几遍sql 都不行



# 解决方法

- 先转换成 varchar 然后在转化成我们想转的其他类型