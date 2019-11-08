# MyBatis 列名不匹配
> 场景: 当我使用一个 JavaBean 对象的属性与数据库列名冲突的时候;reusltType 默认生成resultMap 产生错误;两种解决方案

1. 使用 select 别名
2. 使用 resultMap 元素设置别名

select 方法出现错误:

```xml
	<sql id="allcolumn">
        a_name as "name",password
    </sql>
    <select id="queryAccount" parameterType="string" resultType="account">
        select
         a_name as "name",password as "password"
        from account where "name"=#{root};
    </select>
```

> 最终找到原因:是因为 javabean 属性名与别名相同,需要让别名加双引号,避免冲突
