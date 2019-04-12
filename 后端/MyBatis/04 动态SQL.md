[TOC]<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [动态 SQL](#动态-sql)
- [if](#if)
- [choose,when,otherwise](#choosewhenotherwise)
- [trim,where,set](#trimwhereset)
	- [1. where](#1-where)
	- [2. trim](#2-trim)
- [foreach](#foreach)

<!-- /TOC -->

# 动态 SQL
MyBatis 强大的特性之一通常就是它的动态 SQL 能力.如果你有使用 JDBC 或其他 相似框架的经验,你就明白条件地串联 SQL 字符串在一起是多么的痛苦,确保不能忘了空 格或在列表的最后省略逗号。动态 SQL 可以彻底处理这种痛苦。
动态 SQL 元素和使用 JSTL 或其他相似的基于 XML 的文本处理器相似.MyBatis 采用功能强大的基于 OGNL 的表达式来消除其他元素.

- if
- choose(when,otherwise)
- trim(where,set)
- foreach

# if
在动态 SQL 中所做的最通用的事情是包含部分 where 字句的条件。比如：
```xml
<sql id="allcolumn">
        name,password
</sql>
<select id="findAccount" parameterType="string" resultType="account">
        select
        <include refid="allcolumn"/>
        from account
        where
        name like '%${name}%'
            <if test="password!=null">
               and password like '%${password}%'
            </if>
    </select>
```

> 上述描述了一个姓名模糊匹配，判断密码为非空则添加密码的模糊匹配

# choose,when,otherwise
有时我们不想应用所有的条件，相反我们想选择很多情况下的一种。Java 中的 switch 和语句相似。MyBatis 提供 choose 元素。
```xml
<sql id="allcolumn">
        name,password
</sql>
<select id="findAccount" parameterType="string" resultType="account">
        select
        <include refid="allcolumn"/>
        from account
        where
        name like '%${name}%'
        <choose>
            <when test="password!=null">
                and password like '%${password}%'
            </when>
            <otherwise>
                order by name
            </otherwise>
        </choose>
    </select>
```
> 上述提供类似 switch 的选择元素，when 表示符合的选项，otherwise 元素表示除选项之外的。

# trim,where,set
回到 if 如果是下面的情况，单独存在的 if 拼接的 sql 会产生错误


```xml
<sql id="allcolumn">
        name,password
</sql>
<select id="findAccount" parameterType="string" resultType="account">
    select
    <include refid="allcolumn"/>
    from account
    where
    <if test="name!=null">
        and name like '%${name}%'
    </if>
    <if test="password!=null">
        and password like '%${password}%'
    </if>
</select>
```
> 上述产生错误的 sql 语句：<br>
select name,password<br>
from account<br>
where<br>
and name like '%${name}%'<br>
and password like '%${password}%'

## 1. where
where 元素知道如果包含条件，就添加 where。而且，如果以"AND"或"OR"开头的内容，第一个删除，其他的覆盖
```xml

<select id="findAccount" parameterType="string" resultType="account">
        select
        <include refid="allcolumn"/>
        from account
        <where>
            <if test="name!=null">
               or name like '%${name}%'
            </if>
            <if test="password!=null">
               or password like '%${password}%'
            </if>
        </where>

    </select>
```
也可以通过 trim 解决
## 2. trim

```xml

<select id="findAccount" parameterType="string" resultType="account">
        select
        <include refid="allcolumn"/>
        from account
        <trim prefix="WHERE" prefixOverrides="AND|OR ">
            <if test="name!=null">
               or name like '%${name}%'
            </if>
            <if test="password!=null">
               or password like '%${password}%'
            </if>
        </trim>

    </select>
```

# foreach
另外一个动态 sql 通用的必要操作是迭代一个集合，通常构建在 sql `IN` 条件中

```xml
<select id="findAccount" parameterType="string" resultType="account">
    select
    <include refid="allcolumn"/>
    from account where name IN
    <foreach collection="list" item="item" open="("  separator="," close=")">
       #{item}
    </foreach>
</select>
```

foreach 元素是非常强大的，它允许你指定一个集合，声明集合项和索引变量，它们可以用在元素体内。它也允许你指定开放和关闭的字符串，在迭代之间放置分隔符。

> 注意:你可以传递一个 List 实例或者数组作为参数对象传给 MyBatis。当你这么做的时 候,MyBatis 会自动将它包装在一个 Map 中,用名称在作为键。List 实例将会以“list” 作为键,而数组实例将会以“array”作为键。
