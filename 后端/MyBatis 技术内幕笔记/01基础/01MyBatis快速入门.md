# MyBatis 快速入门

## 1 ORM 介绍

> JDBC是Java与数据库交互的统一API，实际上它分为两组API：
>
> 1. 面向**Java应用程序**开发人员的API
> 2. 面向数**据库驱动程序**开发人员的API

传统的JDBC编程操作查询操作都会设计到重复的步骤，在实践中，为了提高代码的可维护性，可以将重复性代码封装到一个类似`DBUtils`的工具类。其中返回结果集需要完成关系模型到对象模型的转换，要使用比较通用的方式封装这种复杂的转换是比较困难的。因此ORM框架应运而生。

ORM框架的主要功能就是根据**映射配置文件**，完成数据在对象模型与关系模型之间映射，同时也屏蔽了上述重复的代码

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gm4giuhd2pj30rc06qt9k.jpg)

- **应用程序一般需要通过集成缓存、数据源、数据库连接池等组件进行优化**
- **ORM框架一般都提供了集成第三方缓存、第三方数据源等组件的接口方便配置完成第三方组件的集成**

## 2 常见持久化框架

- Hibernate
- JPA
- Spring JDBC (**仅仅是使用模版方式对原声 JDBC 进行了一层非常薄的封装**)
- MyBatis

> MyBatis: 
>
> MyBatis 通过映射配置文件或相应注解配置将ResultSet 映射为Java对象，其映射规则可以嵌套其他映射规则以及子查询，从而实现复杂的映射逻辑，**也可以实现一对一、一对多、多对多映射以及双向映射**，相较于 Hibernate，MyBatis 更加轻量级，可控性也更高。在编写 SQL 语句时，可以比较方便地指定查询返回的列，而不是查询所有列并映射对象后返回，这在列比较多的时候也能起到一定的优化效果
>
> 在实际业务中，对同一数据集的查询条件可能是动态变化的，拼接 SQL 过程非常枯燥、没有技术含量，可能经过反复调试才能得到一个可执行的 SQL 语句。MyBatis 只需要在配置文件中编号动态 SQL 语句，MyBatis 就可以根据执行时传入的实际参数值批凑出完整的、可执行的 SQL 语句

## 3 MyBatis 示例

mybatis-config.xml 配置文件，其中配置了

- 数据库的 URL地址
- 数据库用户名和密码
- 别名信息
- 映射配置文件
- 全局配置信息

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

`BlogMapper.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

Java程序中如何加在上述配置文件如何使用 MyBatis 的API.

- 应用程序首先会加载 `mybatis-config.xml` 配置文件
- 根据配置文件的内容创建 `SqlSessionFactory`对象
- 通过`SqlSessionFactory`对象创建`SqlSession`对象
- `SqlSession`接口中定义了执行 SQL 语句所需要的各种方法
- `SqlSession`对象执行映射配置文件中定义的 SQL 语句,完成相应的数据操作
- 最后通过 `SqlSession`对象提交事务
- 关闭`SqlSession`对象

```java
public static void main(String[] args) {
    String resource = "org/mybatis/example/mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    try (SqlSession session = sqlSessionFactory.openSession()) {
      BlogMapper mapper = session.getMapper(BlogMapper.class);
      Blog blog = mapper.selectBlog(101);
    }
  }
```

也可以使用注解的方式替换 XML 方式

```java
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```

## 4 MyBatis 整体架构

MyBatis 的整体架构分为三层，分别是基础支持层、核心处理层和接口层

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gm4i43zpa0j30ny0fgadf.jpg)

### 1 基础支持层

基础支持层包含整个 MyBatis 的基础模块，这些模块为核心处理层的功能提供良好的支持。

|     模块     | 描述                                                         |
| :----------: | ------------------------------------------------------------ |
|   反射模块   | 反射功能强单，但写出高质量的反射代码还是有一定难度的。       |
| 类型转换模块 | MyBatis 为简化配置文件提供了别名机制，<br />该机制是类型转换模块的主要功能之一。<br />另一个功能是实现 JDBC 类型与 Java 类型之间的转换，<br />该功能在为 SQL 语句绑定实参以及映射查询结果集时都会涉及。 |
|   日志模块   | 优秀的 Log4j、Log4j2、slf4j ，MyBatis 作为一个设计优良的框架，<br />除了提供详细的日志输出信息，还要能够集成多种日志框架，<br />其日志模块的一个主要功能就是集成第三方日志框架 |
| 资源加载模块 | 主要对类加载器进行封装，确定类加载器的使用顺序，<br />并提供了加载类文件以及其他资源文件的功能 |
|  解析器模块  | 解析器模块的主要提供了两个功能：<br />1. 一个功能是对 XPath 进行封装,解析 mybatis-config.xml提供支持<br />2. 为处理动态 SQL 语句中的占位符提供支持 |
|  数据源模块  | 连接池功能、提供连接状态等                                   |
|   事务管理   | MyBatis 对数据库中的事务进行了抽象，其自身提供了相应的事务接口和简单实现 |
|   缓存模块   | 优化数据库性能重要的环节，可以讲部分数据库请求拦截在缓存这一层.MyBatis 中自带的这两级缓存与 MyBatis 以及整个应用是运行在同一个 JVM 中的，共享同一块堆内存。**如果这两级缓存中的数据量较大，则可能影响系统中其他功能的运行，所以当需要缓存大量数据时，优先考虑使用 Redis、Memcache 等缓存产品**/如下图 |
|   Binding    | 在调用 `SqlSession`相应方法执行数据库操作时，需要指定映射文件中定义的 SQL 节点，如果出现拼写错误，我们只能在运行时才能发现相应的异常。为了尽早发现错误，**MyBatis 通过 Binding 模块将用户自定义的`Mapper`接口与映射配置文件关联起来**，系统可以通过调用自定义 Mapper 接口与映射配置文件关联起来。 |

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gm4kjhms03j30em0fqabl.jpg)

### 2 核心处理层

核心处理层中实现了 MyBatis 的核心处理流程，其中包括 MyBatis 的初始化以及完成一次数据库操作的涉及的全部流程

| 模块                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 配置解析                  | MyBatis 初始化过程中，会加载`mybatis-config.xml`配置文件、<br />映射配置文件以及 `Mapper`接口中的注解信息,<br />**解析后的配置信息会形成相应的对象并保存到 Configuration 对象中**。<br />之后，利用 `Configuration`对象创建 `SqlSessionFactory`对象。<br />待初始化之后，可与通过得到 `SqlSessionFactory`创建`SqlSession`<br />对象并完成数据库操作 |
| SQL 解析与 scripting 模块 | MyBatis 实现动态 SQL 语句的功能，提供了多种动态 SQL语句对应的节点，<br />通过这些节点的组合使用，开发人员可以写出几乎满足所有需求的动态 SQL 语句。<br />`scripting`模块会根据用户传入的实参，解析映射文件中定义的动态 SQL 节点，并形成数据库可执行的 SQL 语句。<br />之后会处理 SQL 语句中的占位符，绑定用户传入的实参 |
| SQL执行                   | SQL 语句的执行涉及多个组件，其中比较重要的 Executor、StatementHandler、ParameterHandler 和 ResultSetHandler<br />Executor 主要负责维护一级缓存和二级缓存，并提供事务管理的相关操作，<br />它会将数据库相关操作委托给 `StatementHandler`完成。<br />`StatementHandler`首先通过`ParameterHandler`完成 SQL 语句的实参绑定<br />，然后通过 `java.sql.Statement`对象执行 SQL 语句并得到结果集并返回。如下图 |
| 插件                      | 用户自定义插件可以改变 Mybatis的默认行为，例如，可以拦截 SQL 语句并对其进行重写。由于用户自定义插件会影响 MyBatis 的默认 |

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gm4l61sf8yj30oe0k4juv.jpg)

### 3 接口层

其核心是 `SqlSession`接口，该接口中定义了 MyBatis 暴露给应用程序调用的 API,也就是上层应用与 MyBatis 交互的桥梁。