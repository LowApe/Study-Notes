# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [何为 Maven 坐标](#何为-maven-坐标)
- [坐标详解](#坐标详解)
- [依赖的配置](#依赖的配置)
- [依赖范围](#依赖范围)
- [传递性依赖](#传递性依赖)
	- [何为传递性依赖](#何为传递性依赖)
	- [传递性依赖和依赖范围](#传递性依赖和依赖范围)
- [依赖调解](#依赖调解)
- [可选依赖](#可选依赖)
- [最佳实践](#最佳实践)
	- [排除依赖](#排除依赖)
	- [归类依赖](#归类依赖)
	- [优化依赖](#优化依赖)

<!-- /TOC -->
# 何为 Maven 坐标
Maven 世界拥有庞大数量的构件,不能总是缺一个 jar 就网上寻找,下载,浪费太多时间,还有版本不一致,等等,没有统一的规范,统一的法则,工作就无法自动化.

Maven 定义了这样一组规则:Maven坐标唯一标识.
- groupId
- artifactID
- version
- packaging
- classifier


只要提供正确的坐标元素,Maven 就能找到对应的构件.Maven 从哪里下载构件的?
Maven 内置了一个中央仓库的地址[http://repo1.maven.org/maven2/](http://repo1.maven.org/maven2/),一般需要什么提供正确的坐标,就可以去这个中央仓库下载.

# 坐标详解

- groupId:定义当前 Maven 项目隶属的实际项目
- artifactId:该元素定义实际项目的一个Maven项目(模块),推荐的做法是使用实际项目名称作为前缀
- version:该元素定义 Maven 项目当前所处的版本
- packaging:该元素定义 Maven 项目的打包方式,如:jar;打包方式会影响到构建的生命周期;默认jar;
- classifier:该元素用来帮助定义构建输出的一些附属构件.比如jar,javadoc和sources

> 上面5个,前三个必须,有两个可选,classifier 是不能直接定义的.

> 项目构建的文件名是与坐标相对应的,一般的规则为 `artifactID-version[-classifier].packaging`,其中`[-classifier]`表示可选

# 依赖的配置

`dependecies` 可以包含一个或者多个 dependency 元素,以声明一个或者多个项目依赖.每个依赖可以包含

- groupId,artifactId,version:依赖的基本坐标
- type: 依赖的类型,对于项目坐标定义的 packaging,默认 jar
- scope:依赖的范围
- optional:标记依赖是否可选
- exclusions:用来排除`传递性依赖`

# 依赖范围
JUnit 依赖的测试范围就是 test.首先需要知道,Maven 在编译项目主代码的时候需要使用一套 classpath.如,项目主代码用到 spring-core,该文件以依赖的方式被引入到 classpath.其次, Maven 在编译和执行测试的时候会使用另外一套 classpath.这也是 junit 该文件以依赖的方式引入到测试使用的 classpath 中,不同的是依赖范围是 test.最后,实际运行 Maven 项目的时候,又会使用一套 classpath.

依赖范围就是用来控制依赖与这三种 classpath (编译 classpath,测试 classpath,运行 classpath)的关系,Maven 一下几种依赖范围

- compile:编译依赖范围.默认使用该依赖范围.对于编译,测试,运行三种 classpath 都有效.
- test: 测试依赖范围.只对测试 classpath
- provided: 已提供依赖范围. 前两种有效,运行时无效. 如:servlet-api
- runtime:运行时依赖范围.对于测试和运行 classpath ,但在编译主代码时无效.如: JDBC 驱动
- system:系统范围依赖.与 provided 依赖范围一致.但是,使用 system 范围的依赖必须通过 systemPath 元素显示地指定依赖文件路径.由于此类以来不是通过 Maven 仓库解析的,而且往往与本地系统绑定,肯能构建的不可移植,因此谨慎使用.
- import :导入依赖范围.与三种 classpath 没关系

# 传递性依赖

## 何为传递性依赖
考虑基于 spring framework 的项目,如果不使用 maven,那么项目手动下载相关依赖.但是 spring framework 依赖于其他的开源类库,因此实际会下载很大一包,但是不用那么大,换成小包,如果用到其他依赖还要下载,非常麻烦

Maven 的传递性依赖机制可以很好的解决问题.比如 spring-core 有它自己的依赖,在这个依赖的 pom 包含其他依赖,所使用的范围相同,直接从中央仓库下载该以来,当使用 spring-core 就不考虑它依赖了谁,也不用担心引入多余的依赖. Maven 会解析各个直接的 POM .将那些必要的间接依赖,以`传递性依赖`的形式引入到当前的项目.

## 传递性依赖和依赖范围
依赖范围不仅可以控制依赖与三种 classpath 的关系,还对`传递性依赖`产生影响.
假设A依赖于B，B依赖于C，我们说A对于B是第一直接依赖，B对于C是第二直接依赖，A对于C是传递性依赖

![](http://ww1.sinaimg.cn/large/006rAlqhly1g25cpsmc6zj30or04w3z5.jpg)

上图中间的交叉单元格则表示传递性依赖范围.如果项目有个 `com.icegreen:greenmail:1.3.1b`的直接依赖,我们说这是第一直接依赖,其依赖范围是 test;而 greenmail 又有一个 `javax.mail:mail:1.4` 的直接依赖,我们说这是第二直接依赖,其依赖范围是 compile. 项目与 `javax.mail:mail:1.4` 是传递依赖性,从上图看,一个依赖范围 test 与 第二直接依赖 是 compile 交叉是 test ,因此这个传递性依赖范围是 test;

> 第一直接依赖是表格左边那一列,第二直接依赖是表格第一行.

# 依赖调解

Maven 引入的传递性以来机制,简化和方便了依赖声明,只需关系项目的直接依赖;但那有时候当传递性依赖造成问题的时候,我们就需要清楚地知道该传递性依赖是从哪条依赖路径引入的.

例如,项目 A 有这样依赖关系:`A->B->C->X(1.0)` , `A->D->X(2.0)`,X 是 A 的传递性依赖,但是两条依赖路径上有两个版本的 X,那么哪个 X 会被 Maven 解析使用呢?

> Maven 依赖调解的
- 第一原则:路径最近者优先.
- 第二原则:第一声明者优先.当路径相同,POM 中依赖声明顺序

# 可选依赖
如果 A 依赖与 B , B中依赖两个可选的(其实是不应该使用可选依赖,面向设计原则,单一职责原则)
可选依赖不被传递

# 最佳实践

## 排除依赖
传递性依赖会给项目隐式地引入很多依赖,极大地简化了项目依赖的管理,但是这种特性也会带来问题.例如.当前项目有个第三方依赖,而这个第三方依赖由于某些原因依赖了另外一个类库 snapshot 版本,那么这个 snapshot 就会成为项目的依赖性传递,而 snapshot 的不稳定会直接影响到当前的项目.这时候就需要`排除掉 snapshot` ,并且在当前项目中声明该类库的某个正式发布的版本.还有逆向替换某个传递性依赖,比如 :

> Sun JTA API,Hirbernate 依赖于这个 jar,由于版权的因素,该类库不在中央仓库中,而 Apache Geronimo 项目有个对应的实现.这时你就可以排除 Sun JAT API,再声明 Geronimo 的 JTA API 实现

![](http://ww1.sinaimg.cn/large/006rAlqhly1g25dyjh1ehj30fv0bgad0.jpg)

上面就是一个排除替换的例子.项目A依赖于项目B，但是由于一些原因，不想引入传递性依赖C，而是自己显式地声明对于项目C 1.1.0版本的依赖。代码中使用 exclusions元素声明排除依赖，exclusions 可以包含一个或者多个exclusion子元素，因此可以排除一个或者多个传递性依赖。需要注意的是，声明 exclusion 的时候只需要groupId和 artifactId，而不需要 version 元素，这是因为只需要 groupId 和 artifactId 就能唯一定位依赖图中的某个依赖。换句话说，Maven解析后的依赖中，不可能出现 groupId 和 artifactId 相同，但是 version 不同的两个依赖.

![](http://ww1.sinaimg.cn/large/006rAlqhly1g25e1qxvnaj30c402ddg0.jpg)

## 归类依赖

关于 Soring Framework 的依赖,比如 core,beans,context等等.它们是来自同一项目的不同模块.因此,所有这些依赖的版本都是相同的,而且可以预见,如果升级 Spring Framework 这些依赖的版本会一起升级.类似与java 全局变量,将代码相同出现的提升范围等级,当需要修改的时候,直接修改最大范围的.

```xml
<!-- 定义类似全局变量 -->

<properties>
    <springframework.version>2.5.6</springframework.version>
</properties>


<!-- 依赖用 el 表达式 -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${springframework.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${springframework.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${springframework.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${springframework.version}</version>
    </dependency>
</dependencies>
```

## 优化依赖

在软件开发过程中，程序员会通过重构等方式不断地优化自己的代码，使其变得更简洁、更灵活。同理，程序员也应该能够对Maven项目的依赖了然于胸，并对其进行优化，如`去除多余`的依赖，显式地声明某些必要的依赖

Maven 会自动解析所有项目直接依赖和传递性依赖,并且根据规则正确判断每个依赖的范围,冲突,进行调解确保唯一性.在这些工作之后,最后得到的那些依赖被称为已解析依赖(Resolved Dependency).

```shell
mvn dependency:list
```
> 该命令查看已解析依赖

直接在项目 POM 声明的依赖定义为顶层依赖,而这些顶层依赖则定义为二层,等Maven 依赖解析后,就会构成一个依赖树,通过依赖树就能看清地看到某个依赖路径
```shell
mvn dependency:tree
```

> 或者使用 `dependency:analyze` 工具帮助分析当前项目的依赖,删除 spring-context ,编译,测试,打包都不出错,但是 spring-context-support 依赖 context,使用依赖分析
```shell
mvn dependency:analyze
```

结果:
```shell
[WARNING] Used undeclared dependencies found:
[WARNING]    org.springframework:spring-context:jar:5.1.2.RELEASE:compile
[WARNING] Unused declared dependencies found:
[WARNING]    org.springframework:spring-beans:jar:5.0.12.RELEASE:compile
[WARNING]    org.springframework:spring-core:jar:5.0.12.RELEASE:compile
[WARNING]    ch.qos.logback:logback-classic:jar:1.2.3:compile
```

结果中重要两个部分
- User undeclared dependecies 指项目中使用到的,但是没有显式声明的依赖.这种隐藏的,潜在的危险一旦出现,就往往需要耗费大量的时间来查明真相.

- Unserd declared undeclared dependencies 指项目中未使用的,但显式声明的有 beans 和 core 没用到.不应该简单地删除其声明,而是应该仔细分析.由于 `dependency :analyze` 只会分析编译主代码和测试代码需要用到的依赖,一些执行测试和运行需要的依赖它就发现不了
