# 目录


Maven 的生命周期是抽象的,其实际行为都由插件来完成,如 package 阶段的任务可能就会由 maven-jar-plugin 完成。`生命周期和插件两者协同工作`

# 何为生命周期

在Maven出现之前,项目构建的生命周期就已经存在,软件开发人员每天都在对项目进行`清理`、`编译`、`测试`及`部署`。

maven 的生命周期就是为了对所有的构建过程进行抽象和统一。这个生命周期包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有构建步骤。`也就是说,几乎所有的项目的构建,都能映射到这样的一个生命周期上`

> maven 的生命周期是抽象的,这意味着生命周期本身不做任何实际的工作,在 Maven 的设计中,实际的任务(如编译源代码)都交由插件来完成。

Maven 生命周期抽象了构建的各个步骤,定义了他们的次序,但没由提供具体实现。类似于设计模式中的模板的方法

那具体的代码由谁编写。因此它设计了插件机制。

> 例如:针对`编译`的插件由`Maven-compiler-plugin`,针对`测试`的插件由 `maven-surefire-plugin` 等。当用户由特殊需要的时候,也可以配置插件定制构建行为,甚至自己编写插件。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2iif0p6ecj30k6055wfi.jpg)

# 生命周期详解
## 三套生命周期
- clean(清理项目)
- default(构建项目)
- site(建立项目站点)


每个生命周期包含一些阶段(phase),这些阶段是有顺序的,并且`后面`的阶段依赖于`前面`的阶段用户和 Maven 嘴直接的交互方式就是调用这些生命周期阶段。

clean 生命周期:
1. pre-clean
2. clean
3. post-clean

> 当调用clean 会按照顺序执行 1->2

## clean 生命周期

clean 生命周期的目的是清理项目,它包含三个阶段:
1. pre-clean 执行一些清理前需要完成的工作
2. clean 清理上一次构建生成的文件
3. post-clean 执行一些清理后需要完成的工作

## default 生命周期

default 生命周期定义了真正构建时所需要执行的所有步骤,他是所有生命周期中最核心的部分,其包含的阶段如下,
- validate
- initialize
- generate-sources
- process-source 处理项目主资源文件。一般来说,对 `src/main/resources` 目录的内容进行变量替换等工作后,复制到项目输出的主 classpath 目录中
- generate-reources
- process-resources
- compile 编译项目的主源码。一般来说,是编译 `src/main/java` 目录下的 Java 文件至项目输出的主 classpath 目录中。

- process-classes
- generate-test-resources
- process-test-sources 处理项目测试资源文件。一般来说,是对 `src/test/resources` 目录的内容进行变量替换等工作后,复制到项目输出的测试 classpath 目录中。
- generate-test-resources
- process-test-resources
- test-compile 编译项目的测试代码。一般来说,是编译 `src/test/java` 目录下的 java 文件至项目输出的测试 classpath 目录中
- process-test-classes
- test 使用单元测试框架运行测试,测试代码不会被打包或部署
- prepare-package
- package 接受编译好的代码,打包可发布的格式,如 JAR
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install 将包安装到 Maven 本地仓库,供本地其他 Maven 项目使用
- deploy 将最终的包复制到远程仓库,供其他开发人员和 Maven 项目使用

- 官方文档[http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

## site 生命周期
site 生命周期的目的是建立和发布项目站点,Maven 能够基于 POM 包含的信息,自动生成一个友好的站点,方便团队交流和发布项目信息。 该生命周期包含如下阶段:

- pre-site 执行一些在生成项目站点之前需要完成的工作
- site 生成项目站点文档
- post-site 执行一些在生成项目站点之后需要完成的工作
- site-deploy 将生成的项目站点发布到服务器上。
-
## 命令行与生命周期
常用的 Maven 命令为例,解释其执行的生命周期阶段:
- $ mvn clean:该命令调用 clean 生命周期的 clean 阶段。实际上执行的阶段为 clean 生命周期的 pre-clean 和 clean 阶段

- $mvn test:该命令调用 default 生命周期的 test 阶段。实际执行的阶段为 default 生命周期的 validate 、initialize 等,直到 test 的所有阶段。`这也解释了在执行测试的时候,项目的代码能够自动得以编译`

- $mvn clean install:该命令调用 clean 生命周期的 clean 阶段和 default 生命周期的 install 阶段。实际执行的阶段为 clean 生命周期的 pre-clean、clean 阶段,以及 default 生命周期的从 validate 至 install 的所有阶段。

- $mvn clean deploy site-deploy: 该命令结合了 Maven 所有 三个生命周期,且 deploy 为 default 生命周期的最后一个阶段,site-deploy 为 site 生命周期的最后一个阶段

# 插件目标
对于插件本身,为了能够复用代码,它往往能够完成多个任务。例如 maven-dependecy-plugin,它能够基于项目依赖做很多事情。它能够分析项目依赖,帮助找出潜在的`无用依赖`;它能够列出项目的`依赖树`,帮助分析`依赖源`；为每个这样的功能编写一个独立的插件显然是不可取的,因为这些任务背后有很多可以复用的代码,因此,这些功能聚集在一个插件里,每个功能就是一个`插件目标`。

maven-dependecy-plugin 有十多个目标,每个目标对应一个功能,上述提到的几个功能分别对应的插件目标为

- dependecy:analayz
- depedency:tree
- depedency:list

冒号前面是插件前缀,冒号后面是该插件的目标。类似地,`compiler:compiler`(maven-compiler-plugin的 compile 目标) 和 `surefire:test`(maven-surefire-plugin的test目标)

# 插件绑定

Maven 的生命周期与插件相互绑定,用以完成实际的构建任务。具体而言,是生命周期阶段与插件的目标项目相互绑定,以完成某个具体的构建任务。

例如:项目编译这一任务,它对应了 default `生命周期`的 compile 这一阶段,而 maven-compiler-plugin 这一插件的 compile 目标能够完成该任务。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2in1njofaj30e409b0u1.jpg)

## 内置绑定
Maven 在核心为一些主要的生命周期阶段绑定了很多插件目标,当用户通过命令行调用生命周期阶段的时候,对应插件目标就会执行相应的任务

clean生命周期仅有pre-clean、clean和post-clean三个阶段,其中的clean与maven-clean-plugin:clean绑定。maven-clean-plugin仅有clean这一个目标,其作用就是删除项目的输出目录。

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2inbpk6xbj30df04z0t6.jpg)

site生命周期有 pre-site、site、post-site和site-deploy 四个阶段,其中,site和maven-site-plugin:site相互绑定,site-deploy和maven-site-plugin:depoy相互绑定。maven-site-plugin有很多目标,其中,site目标用来生成项目站点,deploy目标用来将项目站点部署到远程服务器上。

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2ineq2jvtj30dg0503z6.jpg)

由于项目的打包类型会影响构建的具体过程,因此,default 生命周期的阶段与插件目标的绑定关系由项目`打包类型`所决定,打包类型是通过 POM 的 packaging 元素定义的。

基于 jar 类型的项目

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2inj8ud3zj30k506p76j.jpg)


可以通过 Maven 命令行输出中看到在项目构建过程执行了哪些插件目标


```shell
[INFO] ------------------------------------------------------------------------
[INFO] Building SpringP Maven Webapp 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-resources-plugin:3.0.2:resources (default-resources) @ SpringP ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.8.0:compile (default-compile) @ SpringP ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-resources-plugin:3.0.2:testResources (default-testResources) @ SpringP ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory D:\SpringP\src\test\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.8.0:testCompile (default-testCompile) @ SpringP ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-surefire-plugin:2.22.1:test (default-test) @ SpringP ---
[INFO]
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
```

上面是 mven test 命令，可以看出包含生命周期阶段与插件绑定关系
