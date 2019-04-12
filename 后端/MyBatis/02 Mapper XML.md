<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Mapper XML 文件](#mapper-xml-文件)
- [select](#select)
- [insert,update and delete](#insertupdate-and-delete)
- [sql](#sql)
	- [Parameters](#parameters)
	- [字符串替换](#字符串替换)
- [Result Maps](#result-maps)
	- [ResultMap](#resultmap)
		- [id & result](#id-result)
		- [支持的 JDBC 类型](#支持的-jdbc-类型)
		- [构造方法](#构造方法)

<!-- /TOC -->
- [Mapper XML 文件](#mapper-xml-文件)
- [select](#select)
- [insert,update and delete](#insertupdate-and-delete)
- [sql](#sql)
	- [Parameters](#parameters)
	- [字符串替换](#字符串替换)
- [Result Maps](#result-maps)
	- [ResultMap](#resultmap)
		- [id & result](#id-result)
		- [支持的 JDBC 类型](#支持的-jdbc-类型)
		- [构造方法](#构造方法)
		- [关联](#关联)
		- [关联的嵌套 select 查询](#关联的嵌套-select-查询)
		- [关联的嵌套结果映射](#关联的嵌套结果映射)

<!-- /TOC -->
# Mapper XML 文件
MyBatis 核心是映射语句,SQL 映射 XML 语句相当简单.

SQL 映射文件的顶级元素
- `select`
- `insert`
- `update`
- `delete`
- `sql` 可以重用的 SQL 块,也可以被其他语句引用.
- `cache` 配置给定命名空间的缓存
- `cache-ref` 从其他命名空间引用缓存配置
- `resultMap` 最复杂，也是最有力量的元素，描述如何从数据库结果集中加载对象

# select
查询和结果映射
```xml
<select id="menthodName" parameterType="inputType" resultType="returnType">
    select * from account where id=#{id};
</select>
```
> `#{id}` mybatis 创建一个预处理语句参数。使用 jdbc ，这样的一个参数在 sql 中会 "?" 来标识，并将数据传递到这个预处理语句中

> id中如果是通过 `.class` 文件查找，这里的名字需要与类名调用方法保持一致

**我们需要深入了解参数与结果映射**

|属性|描述|
|:--:|:--:|
|id|在命名空间中唯一的标识符,可以被用来引用这条语句|
|parameterType|将会`传入`这条语句的参数类的`完全限定名`或`别名`|
|resultType|从这条语句中返回的期望类型的类的`完全限定名`或`别名`.注意集合情形,集合可以包含类型,而不是集合本身.使用 resultType 或 resultMap,但不能同时使用|
|**resultMap**|命名引用外部的resultMap.返回 map 是 MyBatis 最具力量的特性,对其,重点了解可以让许多复杂映射的情形被结局|
|flushCache|将其设置为 true,无论语句什么时候被调用,都会导致缓存被清空,默认false|
|useCache|导致本条语句结果被缓存.默认:true|
|timeout|等待数据库返回请求结果,并抛出异常时间|
|fetchSize|暗示驱动程序每次批量返回的结果行数,默认不设置|

# insert,update and delete
这三个数据变更语句在实现中类似
```xml
<insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">
```
|属性|描述|
|:--:|:--:|
|statementType|会让MyBatis 选择使用 Statement,PreparedStatement 或 CallableStatement 默认 PreparedStatement|
|useGeneratedKeys|(仅对 insert 有用)这会告诉 MyBatis 使用 JDBC 的getGeneratedKeys 方法来取出由数据内部生成的主键,默认false|
|keyProperty|(仅对 insert 有用)标记一个属性,MyBatis 会通过 getGeneratedKeys 或者通过 insert 语句的 selectKey 子元素设置它的值 默认false|
|keyColumn|(仅对 insert 有用)标记一个属性,MyBatis 会通过 getGeneratedKeys 或者通过 insert 语句的 selectKey 子元素设置它的值 默认false|

```xml
<insert id="insertAuthor" parameterType="domain.blog.Author">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor" parameterType="domain.blog.Author">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>

<delete id="deleteAuthor" parameterType="int">
  delete from Author where id = #{id}
</delete>
```
插入语句有点多,他有一些属性和子属性用来处理主键生成?
首先,如果你的数据库支持`自动生成主键`的字段(比如 MySQL 和 SQL Server) ,那么 你可以设置 useGeneratedKeys=”true”,而且设置 keyProperty 到你已经做好的目标属性上。

```xml
<insert id="insertAuthor"
     parameterType="domain.blog.Author"
     useGeneratedKeys="true"
    <!-- keyProperty="id"> -->
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

MyBatis 有另外一种方法来处理数据库不支持自动生成类型,或者可能 JDBC 驱动不支 持自动生成主键时的主键生成问题。
这里有一个简单(甚至很傻)的示例,它可以生成一个随机 ID(可能你不会这么做, 但是这展示了 MyBatis 处理问题的灵活性,因为它并不真的关心 ID 的生成):

```xml
<insert id="insertAuthor" parameterType="domain.blog.Author">

  <selectKey keyProperty="id"
       resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```
在上面的示例中,selectKey 元素将会首先运行,Author 的 id 会被设置,然后插入语句 会被调用。 这给你了一个简单的行为在你的数据库中来处理自动生成的主键, 而不需要使你 的 Java 代码变得复杂。

selectKey属性

|属性|描述|
|:--:|:--:|
|keyProperty| selectKey 语句结果应该被设置的目标属性|
|resultType|结果类型.MyBatis 通常可以算出来,但是写上也没有问题。 MyBatis 允许任何简单类型用作主键的类型,包括字符串。|
|order|这可以被设置为 BEFORE 或 AFTER.如果设置为 BEFORE,那 么它会首先选择主键, 设置 keyProperty 然后执行插入语句。 如果 设置为 AFTER,那么先执行插入语句,然后是 selectKey 元素- 这和如 Oracle 数据库相似,可以在插入语句中嵌入序列调用。 |
|statementType|和前面的相 同,MyBatis 支持 STA TEMENT ,PREPARED 和 CALLABLE 语句的映射类型,分别代表 PreparedStatement 和 CallableStatement 类型|

# sql
这个元素可以用来定义可重用的 sql 代码段，可以包含在其他语句中

```xml
<sql i="xxx">
    id,username,password
</sql>

<select id="xxxxx" parametertype="User">
    select <include refid="xxx">
    from  table
    where id = #{id}
</select>
```


## Parameters

```xml
<insert id="insertUser" parameterType="User" >
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>

```

`parameterType="User"：命名参数映射`
如果 User 类型的参数对象传递到了语句中, username 和 password 属性将会被查找然后它们的值就被`传递到预处理语句的参数中`。

这点对于传递参数到语句中非常好。但是对于`参数映射`也有一些其他的特性

- mode:允许指定 IN,OUT或INOUT,参数对象属性的真实值会改变(OUT或INOUT)，就像你期望你需要参数的个数。
- javaType:对应 java 类型
- jdbcType:对应 jdbc 类型
- typeHandler:自定义类型处理器类

> 听不懂这一块知识，先暂且搁置

## 字符串替换

- `#{}` 是占位符,然后导致 MyBatis 创建预处理语句,sql 用"?" 表示
- `${}`直接插入不会修改或转义字符

> 重要:接受从用户输出的内容并提供给语句中不变的字符串,这样做是不安全的。这会 导致潜在的 `SQL 注入攻击`,因此你不应该允许用户输入这些字段,或者通常自行转义并检查。

# Result Maps
resultMap 元素是 MyBatis 中最重要最强大的元素,它就是让你远离 90% 的需要从`结果集`中取出数据的 JDBC 不支持的事情,

```xml
<select id="selectUsers" parameterType="int" resultType="hashmap">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

> 这个简单语句`所有列`被自动映射到 HashMap 的`键`上,这由 resultType 属性指定.hashmap 不能很好描述一个`领域模型`.一般使用 JavaBeans,POJOS(Plain Old Java Objects),普通 Java 对象)作为`领域模型`

> 上面 xml 将这三个属性精确匹配到列名.

如果是实体类, MyBatis 在幕后自动创建一个 ResultMap,基于属性名来映射列到 JavaBean 的属性上.如果列名没有精确匹配

1. 列名上你用 select 字句的别名
2. 使用 ResultMap 解决列名不匹配

```xml
<select id="selectUsers" parameterType="int" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```
> 注意:最好别名都用双引号,避免重复

```xml
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="username"/>
  <result property="password" column="password"/>
</resultMap>


<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="username"/>
  <result property="password" column="password"/>
</resultMap>
```
- property:对应实体类名称
- column:对应数据库字段名称

> 注意1:既然使用了别名,property 和 column 就应该不同,负责会出现映射冲突,导致结果不对

> 注意2: 第一个是 resultType 改变成 resultMap;第二个是 select 语句列名与数据库保持相同,where 之后可以使用别名

## ResultMap
- constructor -类在实例化时,用来注入结果到构造方法中
    - idArg ID 参数,标记结果作为 ID 可以帮助提高整体效能
    - arg 注入到构造方法的一个普通结果
- id 一个 id 结构
- result 注入到字段或 JavaBean 属性的普通结果
- assocation 一个复杂的类型关联,许多结果将包成这种类型
    - 嵌入结果映射 结果映射自身的关联
- collection 复杂类型的集
    - 嵌入结果映射 结果映射自身的集
- discriminator 结果值来决定使用哪个结果映射
    - case 基于某些值的结果映射
        - 嵌入结果映射 这种情形结果也映射它本身,因此可以包含很多相同的元素,或者可以参照一个外部的结果映射

一个实例:

```xml
<!-- Very Complex Result Map -->
<resultMap id="detailedBlogResultMap" type="Blog">
  <constructor>
    <idArg column="blog_id" javaType="int"/>
  </constructor>
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
    <result property="favouriteSection" column="author_favourite_section"/>
  </association>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <association property="author" javaType="Author"/>
    <collection property="comments" ofType="Comment">
      <id property="id" column="comment_id"/>
    </collection>
    <collection property="tags" ofType="Tag" >
      <id property="id" column="tag_id"/>
    </collection>
    <discriminator javaType="int" column="draft">
      <case value="1" resultType="DraftPost"/>
    </discriminator>
  </collection>
</resultMap>

```

### id & result

```xml
<id property="id" column="post_id"/>
<result property="subject" column="post_subject"/>
```
这些是结果映射最基本内容.这两者之间的不同是 id 表示的结果将是当比较对象实例时用到的标识属性.这帮助来该进整体表现,特别是`缓存`和`嵌入结果映射`

### 支持的 JDBC 类型
jdbctype:

|BIT|FLOAT|CHAR|TIMESTAMP|OTHER|
|:---:|:---:|:---:|:---:|:---:|
|TINYINT|REAL|VARCHAR|BINARY|BLOG|
|SMALLINT|DOUBLE|LONGVARCHAR|VARBINARY|CLOB|
|INTEGER|NUMERIC|DATE|LONGVARBINARY|BOOLEAN|
|BIGINT|DECIMAL|TIME|NULL|CURSOR|

### 构造方法

构造方法注入允许你在初始化时为类设置属性的值，而不用暴露出公有方法。MyBatis 也支持私有属性和私有 JavaBean 属性来完成注入，但有一些人更青睐于通过构造方法进行注入。 constructor 元素就是为此而生的。

下面是个构造方法:
```java
public class User {
   //...
   public User(Integer id, String username, int age) {
     //...
  }
//...
}
```
将结果注入构造方法,MyBatis 需要通过某种方式定位相应的构造方法.在下面的例子中,MyBatis 搜索一个声明是那个形参的构造方法,参数类型 Integer,String,int ,顺序给出.
```xml
<constructor>
   <idArg column="id" javaType="int"/>
   <arg column="username" javaType="String"/>
   <arg column="age" javaType="_int"/>
</constructor>
```
> 3.4.3 开始,可以指定参数名称的前提下,任意顺序编写,通过添加 `@Param` 注解,或者使用过

```xml
<constructor>
   <idArg column="id" javaType="int" name="id" />
   <arg column="age" javaType="_int" name="age" />
   <arg column="username" javaType="String" name="username" />
</constructor>
```
