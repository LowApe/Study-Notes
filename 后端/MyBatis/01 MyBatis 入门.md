# 目录

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [MyBatis 工作流程](#mybatis-工作流程)
- [使用 XML 中构建 SqlSessionFactory](#使用-xml-中构建-sqlsessionfactory)
- [不使用 XML 中构建 SqlSessionFactory](#不使用-xml-中构建-sqlsessionfactory)
- [从 SqlSessionFactory 中获取 SqlSession](#从-sqlsessionfactory-中获取-sqlsession)
	- [注解映射 sql 语句](#注解映射-sql-语句)
	- [xml 映射 sql 语句](#xml-映射-sql-语句)
- [作用域和生命周期](#作用域和生命周期)
	- [SqlSessionFactoryBuilder](#sqlsessionfactorybuilder)
	- [SqlSessionFactory](#sqlsessionfactory)
	- [SqlSession-过程级](#sqlsession-过程级)
- [参考链接](#参考链接)

<!-- /TOC -->
# MyBatis 工作流程
- 读取配置文件
- 生成 SqlSessionFactory
- 建立 SqlSession
- 调用 MyBatis 提供的API
- 查询 MAP 配置
- 返回结果
- 关闭 SqlSession


# 使用 XML 中构建 SqlSessionFactory

从 XML 文件中构建 SqlSessionFactory 实例 (可以理解为 SqlSessionFactory 获取 xml 各个配置)

> 建议配置 xml 在类路径下的资源文件(resources)进行配置

- 字符串形式的文件路径
- file:// 的url 形式文件路径

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```
- 使用任何输入流实例
- MyBatis 包下的 Resources ,可以从 classpath 或其他位置加载资源文件



# 不使用 XML 中构建 SqlSessionFactory
如果你愿意直接从 Java 代码而不是 XML 文件中创建配置,或者想要创建你自己的配置构建器,MyBatis 也提供完整的配置类,提供所有和 xml 文件相同功能的配置项

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
```

> 其中 BlogDataSourceFactory 是继承SqlSessionFactory,BlogMapper.class 是映射类

# 从 SqlSessionFactory 中获取 SqlSession

```java
SqlSession session = sqlSessionFactory.openSession();
xxx mapper = session.getMapper(xxx.class);
mapper.执行方法();


```
> 上面可以说是从 xml 配置,到执行过程,没有 sql 映射,下面如何让 sql 映射到具体的类呢
    - 注解
    - xml 映射

基本配置文件
 - environments
    - environment
        - dataSource

map 配置文件 xx.XML
 - mappers
    - resource
    - url
    - package


## 注解映射 sql 语句

```java
public interface AccountDao {

    @Select("select name,password from account")
    public List<Account> queryAccount();

    @Insert("insert into account values(#{name},#{password})")
    public int insertAccount(Account account);
}
```

## xml 映射 sql 语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.bolgcode.dao.AccountDao">
    <select id="queryAccount" resultType="com.bolgcode.entities.Account" >
        select name,password from account;
    </select>
    <insert id="insertAccount">
        insert into account(name,password)
        values(#{account.name},#{account.password})
    </insert>
</mapper>
```
> 注意事项:


- xml 需要跟interface dao层名称相同,并且dtd 约束是 mapper 不是 configuration,注意 namespace 是完全限定名
- 注意 mapper 中语句的各种参数
- 上面 select 命名空间`"com.bolgcode.dao.AccountDao"`定义一个名为"queryAccount"映射语句,其实通过"com.bolgcode.dao.AccountDao.queryAccount" 来调用映射语句

# 作用域和生命周期

## SqlSessionFactoryBuilder

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在，以保证所有的 XML 解析资源可以被释放给更重要的事情。

## SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用`单例模式`或者`静态单例模式`。


## SqlSession-过程级
每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例`不是线程安全`的，因此是不能被共享的，所以它的最佳的作用域是`请求`或`方法作用域`。
- 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。
- 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。

如果你现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。 换句话说，每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它。 这个`关闭操作`是很重要的，你应该把这个关闭操作放到 `finally 块`中以确保每次都能执行关闭。


# 参考链接

[mybatis-3](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#Parameters)
