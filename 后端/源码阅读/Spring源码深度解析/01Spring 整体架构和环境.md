# Spring 整体架构和环境搭建

Spring是为了解决企业应用开发的复杂性而创建的。

## Spring整体架构

### Core Container

核心容器包括，**Core、Beans**、Context、Expression Language

**Core：**

- 该模块主要包括 Spring 框架基本的**核心工具**。
- 其他组件都要使用这个包里的类
- 自己的**系统**也使用这些类

**Beans:**

- 访问配置文件
- 创建和管理bean
- Inversion of Controller / DI 相关操作

**Context：**

- 提供类似于 JNDI 注册器的框架式的对象访问方法
- Context 继承于 Beans ，进行相关操作
	- 国际化资源绑定
	- 事件传播
	- 资源加载
- Apllication Context 是 Context 的关键

**Express Language：**

- 强大的表达式语言用于在**运行查询、创建对象**
- JSP 中
	- 支持设置/获取属性的值
	- 属性的分配
	- 方法的调用
	- list 映射
	- list 聚合

### DataAccess / Integration

数据库访问与集成包含有 JDBC、ORM、OXM、JMS 和 Transaction 模块，其中

**JDBC：**

- JDBC 模块提供了一个 JDBC 抽象层，它可以消除编码和解析数据
- 特有的错误代码

**ORM：**

- 对象-关系映射 API，如 JPA、JDO、Hibernate、Mybatis等
- 提供一个交互层

**OXM：**

- 一个对象 Object/XML 映射实现的抽象层

**JMS：**

模块主要包括了一些制造和消费消息的特性。

**Transation：**

支持编程和声明性的事务管理，这些事务类必须实现特定的接口，并且对所有的 POJO 都适用

### Web

Web 上下文模块建立在应用程序上下文模块之上，为基本 Web 应用程序提供了上下文。

**Web模块：**

提供了基础的面向 Web 的集成特性。

- 多文件上传
- 使用 servlet listeners 初始化 IoC 容器以及一个面向 Web 的应用上下文

**Web-Servlet :**

模块 web.servlet.jar，包含Spring 的 model-view-controller 实现。Spring 的 MVC 框架使得模型范围内的代码和 web forms 之间能够清楚地分开

### AOP

提供一个 AOP 联盟标准的面向切面编程的实现，它让你可以定义例如**方法拦截器**和**切点**

- 从而将逻辑代码分开，
- 降低它们之间的耦合性。利用 source-level 的元数据功能

### Test

Test 模块支持使用 JUnit 和 TestNG 对 Spring 组件进行测试



## 环境搭建

gradle 下载(Mac 版本下载)

```shell
brew install gradle
```

从 github 下载 spring-framework

```shell
git clone https://github.com/spring-projects/spring-framework.git
```

导入到 Idea 中

1. File
2. New
3. Project From Existing Source