# 目录

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [MyBatis XML 配置文件](#mybatis-xml-配置文件)
	- [properties:](#properties)
	- [settings](#settings)
	- [typeAliases](#typealiases)
	- [typeHandlers](#typehandlers)
		- [处理枚举类型](#处理枚举类型)
	- [objectFactory](#objectfactory)
	- [plugins](#plugins)
	- [environments](#environments)
		- [TransactionManager](#transactionmanager)
		- [datasource](#datasource)
		- [databaseidprovider](#databaseidprovider)
	- [mappers](#mappers)
- [参考链接](#参考链接)

<!-- /TOC -->


# MyBatis XML 配置文件
配置文件顶层结构:
- configuration（配置）
    - properties（属性）
    - settings（设置）
    - typeAliases（类型别名）
    - typeHandlers（类型处理器）
    - objectFactory（对象工厂）
    - plugins（插件）
    - environments（环境配置）
        - environment（环境变量）
            - transactionManager（事务管理器）
            - dataSource（数据源）
    - databaseIdProvider（数据库厂商标识）
    - mappers（映射器）



## properties:
- 外部引用,替换动态配置属性值,可以有默认值`:`这个特性默认是[关闭的](http://www.mybatis.org/mybatis-3/zh/configuration.html#properties)
- 可以将 properties 值传到 SqlSessionFactory
- properties 加载顺序
    - 指定属性
    - resource 路径下读取属性文件,并覆盖
    - 最后读取作为方法参数传递的属性,并覆盖

```xml
<properties resource="com/bolgcode/dao/db.properties">
    //打开默认
        <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/>
    </properties>
```

```xml
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${db:username?username:ut_user}"/>
</dataSource>
```


## settings
[是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项的意图、默认值等。](http://www.mybatis.org/mybatis-3/zh/configuration.html#settings)

一个配置完整的 settings 元素

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```
## typeAliases

**方法1**
```xml
<typeAliases
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```
**方法2**
```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

```java
@Alias("author")
public class Author {
    ...
}
```
- 使用包名,在该包下的`类的注解名`,当作别名.
- 在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名

> 注意 xml 命名空间中的名称不能使用别名?

java 类型内建相应`类型`别名

|别名|映射类型|
|:--:|:--:|
|_byte|byte|
|_long|long|
|_short|short|
|_int|int|
|_integer|int|
|_double|double|
|_float|float|
|_boolean|boolean|
|string|String|
|byte|Byte|
|long|Long|
|short|Short|
|int|Integer|
|integer|Integer|
|float|Float|
|double|Double|
|等|等|

## typeHandlers

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用`类型处理器`将获取的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。

> 笼统来讲就是 mybatis 传入类型,与返回类型通过 typeHandlers 进行合适的类型转换


****

-  MyBatis 系统的核心设置
    - 数据源(DataSource)
    - 决定事物作用域和控制方式的事物管理器(TransactionManager)


|类型处理器|Java 类型|JDBC 类型|
|:--:|:--:|:---:|
|BooleanType Handler|boolean|数据库兼容的 Boolean|

详细的表[类型处理器(typeHandlers)](http://www.mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)

你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 具体做法为：实现 `org.apache.ibatis.type.TypeHandler` 接口， 或继承一个很便利的类 o`rg.apache.ibatis.type.BaseTypeHandler`， 然后可以选择性地将它映射到一个 JDBC 类型。比如

```java
// ExampleTypeHandler.java
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```
```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

用上述的类型处理器将会覆盖已经存在的处理` Java 的 String 类型`(从继承那看出)属性和 `VARCHAR 参数及结果`(从注解中看出)的类型处理器。 要注意 MyBatis 不会通过窥探数据库元信息来决定使用哪种类型，所以你必须在参数和结果映射中指明那是 VARCHAR 类型的字段， 以使其能够绑定到正确的类型处理器上。这是因为 MyBatis 直到语句被执行时才清楚数据类型。

通过类型处理器的`泛型`，MyBatis 可以得知该类型处理器处理的 Java 类型,不过这种行为可以通过两种方法改变：
- 在类型处理器的配置元素(xml 上的typeHandler 元素) 上增加一个`javaType=String`;
- 在类型处理器的类上(TypeHandler class) 增加一个 `@MappedTypes` 注解来指定与其关联的 Java 类型列表.果在 `javaType` 属性中也同时指定，则注解方式将被忽略。

```java
@MappedJdbcTypes(JdbcType.VARCHAR)
@MappedTypes(value = String.class)
```

可以通过两种方式来指定被关联的 JDBC 类型:
- 在类型处理器的配置元素上增加一个 `javaType` 属性(比如:`jdbcType="VARCHAR"`)
- 在类型处理器的类上增加 `@MappedJdbcTypes` 注解来指定与其关联的 JDBC 类型列表.如果在 `jdbcType` 属性中也同时指定，则注解方式将被忽略。

当在 `ResultMap` 中决定使用哪种类型处理器时，此时 Java 类型是已知的（从结果类型中获得），但是 JDBC 类型是未知的。
此 Mybatis 使用 `javaType=[Java 类型], jdbcType=null` 的组合来选择一个类型处理器。 这意味着使用 `@MappedJdbcTypes` 注解可以限制类型处理器的范围，同时除非显式的设置，否则类型处理器在 `ResultMap `中将是无效的。
如果希望在 `ResultMap` 中使用类型处理器,那么设置 `@MappedJdbcTypes` 注解的 `includeNullJdbcType=true` 即可.
然而从 Mybatis 3.4.0 开始，如果只有一个注册的类型处理器来处理 `Java` 类型，那么它将是 `ResultMap` 使用 `Java` 类型时的默认值（即使没有 `includeNullJdbcType=true`）。
让 MyBatis 为你查找类型处理器:

```java
@MappedJdbcTypes(value = JdbcType.VARCHAR,includeNullJdbcType = true)
```

最后,可以让 mybatis 为你查找类处理器:
```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <package name="org.mybatis.example"/>
</typeHandlers>
```
> 注意:在使用自动发现功能的时候，只能通过注解方式来指定 JDBC 的类型。

你可以创建一个能够处理`多个类`的泛型类型处理器。为了使用泛型类型处理器， 需要增加一个接受`该类的 class` (自定义类)作为参数的构造器，这样在构造一个类型处理器的时候 MyBatis 就会传入一个具体的类。

```javaType
GenericTypeHandler.java
public class GenericTypeHandler <E extends MyObject> extends BaseTypeHandler<E> {
  private Class<E> type;

  public GenericTypeHandler(Class<E> type) {
    if (type == null) throw new IllegalArgumentException("Type argument cannot be null");
    this.type = type;
  }  
```

`EnumTypeHandler` 和 `EnumOrdinalTypeHandler` 都是`泛型`类型处理器，我们将会在接下来的部分详细探讨。

### 处理枚举类型
若想映射枚举类型 `Enum`,则需要从 `EnumTypeHandler` 或者 `EnumOrdinalTypeHandler` 中选一个来使用.

比如说我们想存储近似值时用到的舍入模式.默认情况下,MyBatis 会利用 `EnumTypeHandler` 来把 `Enum` 值转换成对应的名字

> 注意: `EnumTypeHandler` 比较特别,其他的处理器只针对某个特定的类,而他会处理任意继承了 Enum 的类.比如:

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="java.math.RoundingMode"/>
</typeHandlers>
```
 在配置文件中把 `EnumOrdinalTypeHandler` 加到 `typeHandlers` 中即可， 这样每个 `RoundingMode`(或者自定义泛型) 将通过他们的序数值来映射成对应的整形数值。

**怎样将 `Enum` 既映射成字符串又映射成整形呢?**

自动映射器（auto-mapper）会自动地选用 `EnumOrdinalTypeHandler` 来处理， 所以如果我们想用普通的 `EnumTypeHandler`，就必须要显式地为那些 SQL 语句设置要使用的类型处理器。

- [mybaits 映射器](https://github.com/LowApe/Study-Notes/blob/master/%E5%90%8E%E7%AB%AF/MyBatis/03%20%E6%98%A0%E5%B0%84%20XML.md)

## objectFactory
mybatis 每次创建结果对象的新实例时,它都会使用一个对象工厂实例完成.默认的对象工厂需要做的仅仅是实例化目标类,要么通过默认构造方法,要么在参数映射存在的时候通过参数构造方法来实例化.如果想覆盖对象工厂的默认行为,则可以通过创建自己的对象工厂来实现.比如:

```java
// ExampleObjectFactory.java
public class ExampleObjectFactory extends DefaultObjectFactory {
  public Object create(Class type) {
    return super.create(type);
  }
  @Override
     public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
         return super.create(type, constructorArgTypes, constructorArgs);
     }
  public void setProperties(Properties properties) {
    super.setProperties(properties);
  }
  public <T> boolean isCollection(Class<T> type) {
    return Collection.class.isAssignableFrom(type);
  }}
```
> 不知道为什么自己创建会报错
> 解决:类型错了  `List<Class<?>>`

```xml
<!-- mybatis-config.xml -->
<objectFactory type="org.mybatis.example.ExampleObjectFactory">
  <property name="someProperty" value="100"/>
</objectFactory>
```

ObjectFactory 接口很简单，它包含两个创建用的方法，一个是处理默认构造方法的，另外一个是处理带参数的构造方法的。 最后，setProperties 方法可以被用来配置 ObjectFactory，在初始化你的 ObjectFactory 实例后， objectFactory 元素体中定义的属性会被传递给 setProperties 方法。

## plugins

mybatis 允许你在已映射语句执行过程中的某一点进行拦截调用.默认情况下,mybatis 允许使用插件来拦截的方法调用:
- Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
- ParameterHandler (getParameterObject, setParameters)
- ResultSetHandler (handleResultSets, handleOutputParameters)
- StatementHandler (prepare, parameterize, batch, update, query)

这些类中方法的细节可以通过查看每个方法的签名来发现，或者直接查看 MyBatis 发行包中的源代码 **如果你想做的不仅仅是监控方法的调用，那么你最好相当了解要重写的方法的行为。 因为如果在试图修改或重写已有方法的行为的时候，你很可能在破坏 MyBatis 的核心模块。 这些都是更低层的类和方法，所以使用插件的时候要特别当心。**

通过 MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。

```java
// ExamplePlugin.java
@Intercepts({@Signature(
  type= Executor.class,
  method = "update",
  args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
  public Object intercept(Invocation invocation) throws Throwable {
    return invocation.proceed();
  }
  public Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }
  public void setProperties(Properties properties) {
  }
}
```

```xml
<!-- mybatis-config.xml -->
<plugins>
  <plugin interceptor="org.mybatis.example.ExamplePlugin">
    <property name="someProperty" value="100"/>
  </plugin>
</plugins>
```

上面的插件将会拦截在 Executor 实例中所有的 “update” 方法调用， 这里的 Executor 是负责执行低层映射语句的内部对象。

> 上面的插件将会拦截在 Executor 实例中所有的 “update” 方法调用， 这里的 Executor 是负责执行低层映射语句的内部对象。


> **覆盖配置类**
除了用插件来修改 MyBatis 核心行为之外，还可以通过完全覆盖配置类来达到目的。只需继承后覆盖其中的每个方法，再把它传递到 SqlSessionFactoryBuilder.build(myConfig) 方法即可。再次重申，这可能会严重影响 MyBatis 的行为，务请慎之又慎。

## environments

**尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

- 每个数据库对应一个 sqlsessionfactory 实例

为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。可以接受环境配置的两个方法签名是：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, properties);
```
如果忽略了环境参数，那么默认环境将会被加载，如下所示：

```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, properties);
```

环境元素定义了如何配置环境。
```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

- 默认环境 id (default="development")
- 每个 environment 元素定义的环境 id(id="development")
- 事物管理器的配置(type="JDBC")
- 数据源的配置( type="POOLED")

> 可以对环境随意命名，但一定要保证默认的环境 ID 要匹配其中一个环境 ID。

### TransactionManager

在 mybatis 中有两种类型的事务管理器(type="JBDC|MANAGED")

- JDBC 直接使用JDBC 的提交和回滚设置,它依赖于从数据源得到的连接来管理事务作用域.
- MANAGED 这个配置几乎没做什么. 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。

```xml
<transactionManager type="MANAGED">
  <property name="closeConnection" value="false"/>
</transactionManager>
```

> 注意:如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器， 因为 Spring 模块会使用自带的管理器来覆盖前面的配置。


使用这两个接口，你可以完全自定义 MyBatis 对事务的处理。

```java

public interface Transaction {
  Connection getConnection() throws SQLException;
  void commit() throws SQLException;
  void rollback() throws SQLException;
  void close() throws SQLException;
  Integer getTimeout() throws SQLException;
}

public interface TransactionFactory {
  void setProperties(Properties props);
  Transaction newTransaction(Connection conn);
  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit);
}

```
### datasource
dataSource 元素使用标准的 jdbc 数据源接口配置 jdbc 连接对象的资源.

- 许多 MyBatis 的应用程序会按示例中的例子来配置数据源。虽然这是可选的，但为了使用`延迟加载`，数据源是必须配置的。

三种数据源类型

1. unpooled
2. pooled
3. jndi

**unpooled**
这个数据源的实现只是每次被请求时打开和关闭连接。虽然有点慢，但对于在数据库连接可用性方面没有太高要求的简单应用程序来说，是一个很好的选择。 不同的数据库在性能方面的表现也是不一样的，对于某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形
UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

- dirver
- url
- useranme
- password
- defaultTransactionIsolationLevel 默认的连接事物隔离级别.

作为可选项，你也可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：

- driver.encoding=UTF8
这将通过 `DriverManager.getConnection(url,driverProperties)` 方法传递值为 UTF8 的 encoding 属性给数据库驱动。

**pooled**
这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了`创建新的连接实例`时所必需的初始化和认证`时间`。 这是一种使得`并发` Web 应用快速响应请求的流行处理方式。

这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这是一种使得并发 Web 应用快速响应请求的流行处理方式。

除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

- `poolMaximumActiveConnections` – 在任意时间可以存在的活动（也就是正在使用）连接数量，默认值：10
- `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
- `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
- `poolTimeToWait` – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直安静的失败），默认值：20000 毫秒（即 20 秒）。
- `poolMaximumLocalBadConnectionTolerance` – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 `poolMaximumIdleConnections 与 poolMaximumLocalBadConnectionTolerance` 之和。 默认值：3 （新增于 3.4.5）
- `poolPingQuery` – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。
- `poolPingEnabled` – 是否启用侦测查询。若开启，需要设置 poolPingQuery 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
 `poolPingConnectionsNotUsedFor` – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

**jndi**

这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。这种数据源配置只需要两个属性：

- `initial_context`这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。

- `data_source`这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

> flag : 这块没看懂

### databaseidprovider
[databaseidprovider 文档](http://www.mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)

> flag:目前每接触到

## mappers


```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>

<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>


<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>


<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

# 参考链接

[mybatis-3](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#Parameters)
