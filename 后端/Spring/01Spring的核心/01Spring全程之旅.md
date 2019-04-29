# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [Spring 概述](#spring-概述)
- [简化 Java 开发](#简化-java-开发)
	- [激发 POJO 的潜能???](#激发-pojo-的潜能)
	- [依赖注入](#依赖注入)
		- [观察它如何工作](#观察它如何工作)
	- [应用切面](#应用切面)
	- [使用模版消除样板式代码](#使用模版消除样板式代码)
- [容纳你的 Bean](#容纳你的-bean)
	- [使用应用上下文](#使用应用上下文)
	- [bean的生命周期](#bean的生命周期)
- [宏观看 Spring](#宏观看-spring)
	- [Spring 模块](#spring-模块)
		- [Spring 核心容器](#spring-核心容器)
		- [Spring 的AOP模块](#spring-的aop模块)
		- [数据访问与集成](#数据访问与集成)
		- [Web 与远程调用](#web-与远程调用)
		- [Spring Portfolio](#spring-portfolio)
		- [Spring Web Flow](#spring-web-flow)
		- [Spring Web Service](#spring-web-service)
		- [Spring Security](#spring-security)
		- [Spring Integration](#spring-integration)
		- [Spring Batch](#spring-batch)
		- [Spring Social](#spring-social)
		- [Spring Mobile](#spring-mobile)
		- [Spring for Android](#spring-for-android)
		- [Spring Boot](#spring-boot)
- [Spring 的新功能](#spring-的新功能)

<!-- /TOC -->
# Spring 概述
Spring 可以做许多事情,为企业级开发提供丰富的功能,这些功能底层都依赖于两个核心特性`dependency injection(依赖注入DI)`和`aspect-oriental programming(面向切面的编程AOP)`.

> 下面是整个 Spring 整体概述,比较多的概念,后几章才是实战.不过了解整体结构有助于学习.
# 简化 Java 开发
Spring 是为了解决企业级应用开发的复杂性而创建的.
Spring 如何全方位的简化 Java 开发?

- 基于 POJO 的轻量级和最小侵入性编程
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模版减少样模版式代码

## 激发 POJO 的潜能???
很多框架通过强迫应用继承它们的类或实现它们的接口从而导致应用与框架绑死.典型例子[EJB](https://zh.wikipedia.org/wiki/EJB)2时代的无状态会话bean.这种`侵入式`的编程方式在早期版本的 `Struts,WebWork,Tapestry`无数Java规范和框架中看到

Spring 不会强迫你继承Spring规范的类,相反它的类通常没有任何痕迹表明你使用了 Spring.最坏的场景是一类或许会使用Spring 注解,但它依旧是 [POJO](https://baike.baidu.com/item/POJO).比如:

```java
package top.blogcode.spring;
public class HelloWorldBean{
    public String sayHello(){
        return "Hello World"
    }
}
```

> 这是简单普通的Java类(POJO).没有任何地方表明它是一个Spring组件.Spring的`非侵入`编程模型意味着这个类在Spring应用和非Spring应用中都可以发挥同样的作用.Spring 赋予 POJO 魔力 的方式之一就是通过 DI 来装配它们.

## 依赖注入
在项目中应用 DI ,你会发现你的代码会变的异常简单并且更容易理解和测试.

**DI功能如何实现**
任何一个有实际意义的应用都会由两个或者更多的类组成,这些类相互之间进行协作来完成特定的业务逻辑.按照传统的做法,每个对象负责管理与自己相互协作的对象(所依赖的对象)的引用,这会导致`高度耦合`和`难以测试代码`.例如:

```java
package top.blogcode.spring;
public class HelloWorldBean{
    private Person person;
    public HelloWorldBean(){
        this.person=new Person();
    }

    public void runPersonMethod(){
        person.say();
    }
}
```

> 上面 HelloWorldBean 的构造方法中创建了 Person ,到这类与类之间耦合到一起,为这个 HelloWorldBean 编写单元测试非常地困难.再这样的一个测试中,你必须保证调用 runPersonMethod() 方法同时,say() 的方法也能够实现.但是没有一个简明的方式能够实现.

耦合具有两面性,
- 一方面,紧密耦合的代码难以测试,难以复用,难以理解,且典型地表现出现"打地鼠"式的 bug 特性(修一个bug,出现一个或多个bug)
- 另一方面,一个程度的耦合又是必须的.完全没有耦合的代码什么什么也做不了.

总而言之,耦合是必须的,但应当被小心谨慎地管理.
通过DI,对象的依赖关系将有`系统中`负责协调各对象的第三方组件在创建对象的时候设定.对象无需自行创建或管理他们的依赖关系,如图:

![](http://ww1.sinaimg.cn/large/006rAlqhly1g237bppv7lj30an05xwee.jpg)

```java
package top.blogcode.spring;
public class HelloWorldBean{
    private Person person;
    public HelloWorldBean(Person person){
        this.person=person;
    }

    public void runPersonMethod(){
        person.say();
    }
}
```

不同于之前,Person 没有自行创建,而是在构造的时候作为构造器参数传入.这是依赖注入的方式之一,即构造器注入(constructor injection)

> 传入的类型是 Person,也就是所有任务都必须实现实现一个`接口`(Person 抽象成接口),可以是实现了 Person 的任意衍生类,这里没有与这些衍生类发生耦合.这就是 DI 所带来的最大好处`松耦合`,**如果一个对象中通过接口来表明依赖关系,那么这种依赖就能够在对象本身毫不知情的情况下,用不同的具体实现进行替换.**

创建应用组件之间协作行为叫装配(wiring),String 有多种装配 bean 的方式,采用 xml 是一种常见装配方式


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="com.springinaction.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="com.springinaction.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

</beans>

```
> 上面是书上的配置

![](http://ww1.sinaimg.cn/large/006rAlqhly1g239msopqmj30o50ebq4h.jpg)

还有一种使用 Java 来描述配置.

```java
package com.springinaction.knights.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.springinaction.knights.BraveKnight;
import com.springinaction.knights.Knight;
import com.springinaction.knights.Quest;
import com.springinaction.knights.SlayDragonQuest;

@Configuration
public class KnightConfig {

  @Bean
  public Knight knight() {
    return new BraveKnight(quest());
  }

  @Bean
  public Quest quest() {
    return new SlayDragonQuest(System.out);
  }

}
```

### 观察它如何工作

Spring 通过应用上下文(ApplicationContext)装载bean,并把它们组装起来.这个 ApplicationContext 全权负责对象的创建和组装.Spring 自带了多种上下文的实现,区别仅仅在于如何加载配置

- ClassPathXmlApplicationContext 适合 bean 用xml文件配置

简单demo
```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-config.xml");
        Knight user = (Knight) applicationContext.getBean(Knight.class);
```

## 应用切面

 DI 能够让相互协作的软件组件保存松散耦合,而面向切面编程(aspect-oriented programming,AOP) 允许你把遍布应用各处的功能分离出来形成**可重用的组件**.

**面向切面编程**:一个系统由不同的组件构成,但是这些组件除了负责特定功能,还经常承担额外的职责(日志,事务,管理和安全),这些系统服务通常称为"横切关注点(跨域问题)",因为它们会跨越系统多个组件.

如果将这些关注点分散到多个组件中,代码将会带来双重复杂性.

- 实现系统`关注点功能`的代码将会重复出现多个组件中(即时将除核心代码部分抽象出一个独立模块,然后调用,也还是重复在各个模块中)
- 组件会因为那些与自身核心业务无关的代码而变得混乱

![](http://ww1.sinaimg.cn/large/006rAlqhly1g23an03vkoj30h308aq4x.jpg)
上图展现了系统服务过于紧密

AOP 能够使这些服务模块化,并以声明的方式将它们应用需要影响的组件中去.所造成的结果就是这些组件会具有更高的内聚性并且会更加关注自身的业务,完全不需要了解涉及系统服务所带来复杂性.总之,AOP 能确保 POJO的简单性.

![](http://ww1.sinaimg.cn/large/006rAlqhly1g23avvzfxrj30dw091ab6.jpg)

看一下书上的例子:讲的是一个诗人类处理骑士类的一个事件,在骑士类之前,之后分别处理
```java
package com.springinaction.knights;

public class BraveKnight implements Knight {

  private Quest quest;
  private Minstrel minstrel;

  public BraveKnight(Quest quest, Minstrel minstrel) {
    this.quest = quest;
    this.minstrel = minstrel;
  }

  public void embarkOnQuest() throws QuestException {
    minstrel.singBeforeQuest();
    quest.embark();
    minstrel.singAfterQuest();
  }

}

```
> 其中骑士类除了自己的核心代码,还传入了一个诗人类, 利用 AOP ,你可以声明诗人类必须处理方法,但是骑士本身并不用直接访问诗人的方法.所有将诗人抽象为一个切面,现在需要在 xml 将这个类声明为一个切面.

```xml
<?xml version="1.0" encoding="UTF-8"?>>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop
      http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
    http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="knight" class="com.springinaction.knights.BraveKnight">
    <constructor-arg ref="quest" />
  </bean>

  <bean id="quest" class="com.springinaction.knights.SlayDragonQuest">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <bean id="minstrel" class="com.springinaction.knights.Minstrel">
    <constructor-arg value="#{T(System).out}" />
  </bean>

  <aop:config>
    <aop:aspect ref="minstrel">
      <aop:pointcut id="embark"
          expression="execution(* *.embarkOnQuest(..))"/>

      <aop:before pointcut-ref="embark"
          method="singBeforeQuest"/>

      <aop:after pointcut-ref="embark"
          method="singAfterQuest"/>
    </aop:aspect>
  </aop:config>

</beans>

```
- 把 minstrel 声明为一个 bean
- 然后再 <aop:aspect> 元素引用这个 bean
- <aop:before> 在 embarkOnQuest() 方法执行前调用用 Minstrel 的 singBeforeQuest()方法.这种方式称为前置通知.
- <aop:after> 在 embarkOnQuest() 之后执行singAfterQuest()方法.这种方式称为后置通知.

这两种方式中, `pointcut-ref` 属性都引用名为 embark 的切入点.该切入点在之前<pointcut> 元素中定义的,并配置 expreesion 属性来选择应用的通知.表达式语法采用 `AspectJ` 的切点表达式语言.

Minstrel 可以被应用到 BraveKnight 中,而 BraveKnight 不需要显示地调用它.实际上,`BraveKnight 完全不知道 Minstrel 的存在`

## 使用模版消除样板式代码
以前总会感觉以前曾经写过一些相同的代码?这就是`样板式代码`(boilerplate code).样板式代码常见例子就是 JDBC 访问数据库查询数据

```java
public Employee getEmployeeById(long id) {
  Connection conn = null;
  PreparedStatement stmt = null;
  ResultSet rs = null;
  try {
    conn = dataSource.getConnection();
    stmt = conn.prepareStatement(
          "select id, firstname, lastname, salary from " +
          "employee where id=?");
     stmt.setLong(1, id);
    rs = stmt.executeQuery();
    Employee employee = null;
    if (rs.next()) {
      employee = new Employee();
      employee.setId(rs.getLong("id"));
      employee.setFirstName(rs.getString("firstname"));
      employee.setLastName(rs.getString("lastname"));
      employee.setSalary(rs.getBigDecimal("salary"));
    }
    return employee;
  } catch (SQLException e) {
  } finally {
      if(rs != null) {
        try {
          rs.close();
        } catch(SQLException e) {}
      }
      if(stmt != null) {
        try {
        stmt.close();
        } catch(SQLException e) {}
      }
      if(conn != null) {
        try {
          conn.close();
        } catch(SQLException e) {}
      }
  }
  return null;
}

```

Spring 通过模版封装来消除样板式数据库操作时,避免传统的 JDBC 样板代码.

# 容纳你的 Bean
在基于 Spring 的应用中,你的应用对象生存于 Spring 容器(container) 中,Spring 容器负责创建对象,装配它们,配置它们并创建它们的整个生命周期,从 new 到 finalize()

Spring 自带多容器实现,可以归为两种不同的类型
- bean 工厂(由`org.springframework,beans.factory.BeanFactory`接口定义)
- 应用上下文(由`org.springframework.context.ApplicationContext接口定义`)基于BeanFactory 构建.最常用的一种方式

## 使用应用上下文

- `AnnotationConfigApplicationContext`:从一个或多个基于 Java 配置类中加载 Spring 应用上下文
- `AnnotationConfigWebApplicationContext`:从一个或多个基于 Java 的装配类中加载Spring Web 应用上下文.

- `ClassPahtXmlApplicationContext`:从类路径下的一个或多个 XML 配置文件中加载上下定义,把应用上下文的定义文件作为类资源
- `FileSystemXmlapplicationContext`:从文件系统下的一个或多个 XML 配置文件中加载上下文定义
- `XmlWebApplicationContext`:从Web应用下的一个或多个 XML 配置文件中加载.

## bean的生命周期

![](http://ww1.sinaimg.cn/large/006rAlqhly1g23dhqmu39j30j809jwhx.jpg)
根据上图步骤:

1. Spring 对 bean 进行实例化
- Spring 将`值`和 `bean` 的引用注入到 bean 的对应的属性
- 如果 bean 实现 `BeanNameAware` 接口,Spring 将bean 的 ID 传递给 setBeanName()方法
- 如果 bean 实现 `BeanFactoryAware` 接口,Spring 将调用 setBeanFactory() 方法,将 BeanFactory 容器实例传入
- 如果 bean 实现 `ApplicationContextAware` 接口,Spring 将调用 setApplicationContext()方法,将bean所在的应用上下文的引用传入进来
- 如果 bean 实现 `BeanPostProcessor` 接口,Spring 将调用ProcessBeforeInitialization()方法;
- 如果 bean 实现 `InitializingBean` 接口,Spring 将调用afterPropertiesSet()方法.类似地,如过 bean 使用 init-method 声明了初始化方法,该方法也会被调用
- 如果 bean 实现 `BeanPostProcessor` 接口,Spring 将调用ProcessAfterInitialization()方法;
- 此时,bean 已经准备就绪,可以被应用程序使用了,它们将一直驻留在应用上下文中,直到该应用上下文被销毁
- 如果 bean 实现 DisposableBean 接口,Spring将调用它的 destory()接口方法.同样,如果 bean 使用 destory-method 声明了销毁方法,该方法会被调用.

# 宏观看 Spring

目前为止:Spring 框架关注于
1. DI
2. aop
3. 消除样板式代码简化JavaEE开发

在 Spring 框架之外还存在一个构建在核心架构之上的庞大生态圈,它将 Spring 扩展到不同的领域,例如 Web 服务,REST,移动开发以及 NoSQL


## Spring 模块
下载Spring4.0发布版本并查看其 lib 目录,分为 20 多个不同模版,每个模版有 3 个 jar 文件(二进制类库,源码的jar文件以及JavaDoc 的jar 文件). 目前[Spring5.0](https://spring.io/projects) 提供22种

![](http://ww1.sinaimg.cn/large/006rAlqhly1g23eulchpyj30kc0ds42w.jpg)

> 六个定义良好的模块分类组成

### Spring 核心容器

容器是 Spring 框架最核心的部分,它管理着 Spring 应用中 bean 的创建,配置和管理.在该模块中,包括了 Spring bean 工厂,它为 Spring 提供了 DI 的功能.基于 bean 工厂,我们还会发现有多种 Spring 应用上下文的实现,每一种都提供了配置 Spring 的不同方式.

所有的Spring模块都构建于`核心容器`之上.当你配置应用时,其实你隐式地使用了这些类

### Spring 的AOP模块

在 AOP 模块,Spring 对面向切面编程提供了丰富的支持.这个模块时 Spring 应用系统中开发切面的基础.与 DI 一样,AOP 可以帮助应用对象解耦.借助于 AOP ,可以将遍布系统的关注点(事务和安全)从它们所应用的对象中解耦出来.

### 数据访问与集成
JDBC 通常会导致大量的样板代码,从获取连接,创建语句,处理结果集到最后关闭.Spring 的 JDBC 和 DAO(Data Access Object) 模块抽象了这些样板式代码,使我们的数据库代码变得简单明了,还可以避免因为关闭数据库资源失败而引发的问题. 该模块在多种数据库服务的错误信息之上构建了一个语义丰富的异常层,不需要解释SQL 错误信息.

ORM(Object-Realational Mapping)模块建立在对 DAO 的支持之上,并为多个 ORM 框架提供了一种构建 DAO 的简便方式.

ORM 对多种流行的 ORM 框架进行了集合,包括 Hibernate,Java Persisiternce API,Java Data Object 和 IBatis SQL Maps.

### Web 与远程调用
MVC 模式,虽然 Spring 能够与多种流行的 MVC 框架进行集成,但它的 Web和 远程调用模块自带了一个强大的 MVC 框架,有助于在 Web 层提升应用的松耦合水平

### Spring Portfolio
Spring Portfolio 为每个领域的 Java 开发都提供了 Spring 编程模型

### Spring Web Flow
Spring Web Flow建立于Spring MVC框架之上，它为基于流程的`会话式Web`应用(可以想一下购物车或者向导功能)提供了支持。你还可以访问 [Spring Web Flow的主页](http://projects.spring.io/spring-webflow/)

### Spring Web Service
虽然核心的Spring框架提供了将Spring bean以声明的方式发布为Web Service的功能，但是这些服务是基于一个具有争议性的架构(拙劣的契约后置模型)之上而构建的。这些服务的契约由bean的接口来决定。Spring Web Service 提供了契约优先的 Web Service 模型,服务的实现都是为了满足服务的契约而编写的.

### Spring Security
安全对于许多应用都是一个非常关键的切面。利用Spring AOP，Spring Security为Spring应用提供了声明式的安全机制。

### Spring Integration

许多企业级应用都需要与其他应用进行交互.Spring Integration 提供了多种通用应用`集成模式` [相关连接](https://spring.io/projects/spring-integration)

### Spring Batch
当我们需要对数据进行大量操作时，没有任何技术可以比`批处理`更胜任这种场景。如果需要开发一个批处理应用，你可以通过Spring Batch，使用Spring强大的面向POJO的编程模型。


### Spring Social
社交网络是互联网领域中新兴的一种潮流,越来越多的应用正在融入社交网络网站

### Spring Mobile
### Spring for Android
### Spring Boot
Spring 极大地简化了众多的编程任务，减少甚至消除了很多样板式代码，如果没有Spring的话，在日常工作中你不得不编写这样的样板代码。Spring Boot是一个崭新的令人兴奋的项目，**它以Spring的视角，致力于简化Spring本身。**

Spring Boot 大量依赖于`自动配置`技术，它能够消除大部分(在很多场景中，甚至是全部Spring配置。它还提供了多个Starter项目，不管你使用Maven还是Gradle，这都能减少 Spring 工程构建文件的大小.

# Spring 的新功能
> 书上3.1,3.2,4.0 ,目前最新 5.0
