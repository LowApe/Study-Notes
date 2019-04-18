# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [编写POM](#编写pom)
- [编写主代码](#编写主代码)
- [编写测试代码](#编写测试代码)
- [打包和运行](#打包和运行)
- [使用 Archetype 生成项目骨架](#使用-archetype-生成项目骨架)
- [Idea 下使用 Maven](#idea-下使用-maven)

<!-- /TOC -->
# 编写POM
就像Make的Makefile,Ant的build.xml一样,Maven项目的核心是`pom.xml`.POM(Project Object Model,项目对象模型)定义了项目的基本信息,用于描述项目如何构建,声明项目依赖,等等.现在编写一个HelloWorld

- 创建文件夹
- 新建 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.blogcode.test01</groupId>
    <artifactId>hello-world</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>Maven Hello World Project</name>
</project>

```

- 第一行是xml头,指定了该xml文档的版本和编码方式
- project 是 pom.xml 的根元素,它还声明一些pom相关的命名空间及xsd元素,虽然这些属性不是必须的,但使用这些属性能够让第三方工具(如 IDE 中 XML 编辑器)帮助我们快速编辑 POM.

根元素下的第一个子元素 modelVersion 指定当前pom模型的版本,对于 Maven 2 及 Maven 3 来说,它只能是4.0.0

这段代码中最重要的是包含groupId,artifactId和version 的三行.这三个元素定义了一个基本项目的坐标,任何的jar,pom或者war都是基于这些基本的坐标进行区分

- groundId 定义了项目属于哪个组,这个组往往和项目所在的组织或公司存在关联.
- artifactId 定义了当前 Maven 项目在组中唯一的ID,我们为hello-world
- version hello world 项目当前的版本 1.0-snapshot,表示快照,说明项目处于开发中,是不稳定的版本.

- name 元素声明一个对与用户更好的项目名称,虽然这不是必须的,但还是推荐为每个 pom 声明 name,以方便信息交流

> 没有任何实际的 Java 代码,我们就能够定义一个 Maven项目的 POM ,这体现了 Maven 的一大优点,它能让项目对象模型最大程度地与实际代码相独立,我们可以称之为`解耦`.
    - 如果项目升级版本,只需要修改POM,而不需要更改 Java 代码
    - POM 稳定之后,日常的 Java 代码开发工作基本不修改 POM

# 编写主代码
项目主代码和测试代码不同,项目的主代码会被打包到最终的构件中(如jar),而测试代码只在运行测试时用到,不会被打包.默认情况下,Maven 假设项目主代码位于 `src/main/java`(Maven 的约定)目录,然后在该目录下创建文件 `com/xxx/xxx/projectName/HelloWorld.java`,并编写 Java 代码

项目的包应该与 groupID 和 artifactId 相吻合.这样更加清晰,更加符合逻辑,也方便搜索构件或者 Java 类.

使用 Maven 在项目根目录运行 `mvn clean compile`

```shell
E:\Hello_World>mvn clean compile
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------< com.blogcode.helloworld:Hello_World >-----------------
[INFO] Building Hello_World 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ Hello_World ---
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ Hello_World ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory E:\Hello_World\src\main\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ Hello_World ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to E:\Hello_World\target\classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.507 s
[INFO] Finished at: 2019-04-14T14:23:35+08:00
[INFO] ------------------------------------------------------------------------
```

- clean 告诉 Maven 清理输出目录 `target/`,compile 告诉 Maven 编译项目主代码,从输出中看到 Maven 首先执行了 clean:clean 任务,删除 target/目录;
- 接着执行 resources:resources 任务(未定义项目起源,略过);最后执行
- compiler:compile任务,将项目主代码编译至 `traget/classes`目录(编译好的类为`com\bolgcode\hellworld\HelloWorld.Class`)

前面提到的 clean:clean,resources:resources 和 compiler:compile 对应一些 Maven 插件及插件目标:
- clean:clean是clean插件的clean 目标
- compiler:compile 是 compiler 插件的 compile 目标.

# 编写测试代码

为了使项目保持清晰,主代码与测试代码应该分别位于独立的目录中.Maven 默认测试代码目录 `src/test/java`.

在 Java 中,有 Kent Beck 和 Erich Gamma 建立的 JUnit 是事实上的单元测试标准.要使用 JUnit,首先需要为 Hello World 项目添加一个 JUnit 依赖,在 POM 中添加.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.blogcode.helloworld</groupId>
    <artifactId>Hello_World</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.7</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

```
有了这段声明,Maven 能够自动下载 junit-4.7.jar.Maven 之前,可以去junit 官方网站下载分发包,有了 Maven,它会自动访问`中央仓库` [http://repo1/maven.org/maven2/](http://repo1/maven.org/maven2/),下载需要的文件.读者也可以自己访问该仓库.

上面还有一个值为 test 的元素 scope,scope 为`依赖范围`,若依赖范围为test则表示该依赖只对测试有效.换句话说,测试代码中的 import JUnit 代码是没有问题的,但是如果在主代码中用 import JUnit 代码,就会造成编译错误

```java

package com.bolgcode;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class HelloWorld{
    @Test
    public void testsayHello(){
        HelloWorld helloWorld = new HelloWorld();
        String result  = helloWorld.sayHello();
       assertEquals("Hello Maven",result);
    }
}
```

一个典型的单元测试包含三个步骤:
1. 准备测试类及数据
2. 执行要测试的行为
3. 检查结果
最后使用了 JUnit 框架的 Assert 类检查结果是否为我们期望的 "Hello Maven".

> 在 JUnit3 中,约定所有需要执行测试的方法都以 test 开头,这里使用了 JUnit 4 ,但仍然遵循这一约定. 执行测试方法都应该以 @Test 进行标注

测试用例编写完毕后,可以使用 mvn clean test 进行测试.


```shell
E:\Maven_Study>mvn clean test
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------< com.blogcode.helloworld:Hello_World >-----------------
[INFO] Building Hello_World 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
Downloading from alimaven: http://maven.aliyun.com/nexus/content/groups/public/junit/junit/4.7/junit-4.7.pom
Downloaded from alimaven: http://maven.aliyun.com/nexus/content/groups/public/junit/junit/4.7/junit-4.7.pom (1.2 kB at 1.7 kB/s)
Downloading from alimaven: http://maven.aliyun.com/nexus/content/groups/public/junit/junit/4.7/junit-4.7.jar
Downloaded from alimaven: http://maven.aliyun.com/nexus/content/groups/public/junit/junit/4.7/junit-4.7.jar (232 kB at 481 kB/s)
[INFO]
```
虽然 Maven 执行了 test,而Maven实际执行却不止这两个

```shell
E:\Hello_World>mvn clean test
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------< com.blogcode.helloworld:Hello_World >-----------------
[INFO] Building Hello_World 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ Hello_World ---
[INFO] Deleting E:\Hello_World\target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ Hello_World ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory E:\Hello_World\src\main\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ Hello_World ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to E:\Hello_World\target\classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ Hello_World ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory E:\Hello_World\src\test\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ Hello_World ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to E:\Hello_World\target\test-classes
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ Hello_World ---
[INFO] Surefire report directory: E:\Hello_World\target\surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.bolgcode.HelloWorldTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.042 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.089 s
[INFO] Finished at: 2019-04-14T14:24:20+08:00
[INFO] ------------------------------------------------------------------------
```

# 打包和运行

将项目进行编译,测试之后,下一个重要步骤就是打包类型,默认jar.执行 mvn clean package 进行打包,

```shell

[INFO]
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ Hello_World ---
[INFO] Building jar: E:\Hello_World\target\Hello_World-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.107 s
[INFO] Finished at: 2019-04-14T14:25:17+08:00
[INFO] ------------------------------------------------------------------------
```
Maven 会在打包之前执行编译,测试等操作.这里看到jar:jar任务负责打包,将项目主代码打包成一个名为 `Hello-World-1.0-SNAPSHOT.jar`,位于traget/目录.它是根据 artifact-version. jar 规则进行命名的,如有需要,还可以使用 finalName 来定义该文件的名称,这里暂且不展开.

如何才能让其他 Maven 项目直接引用这个 jar ? 执行 `mvn clean install`

> 打包之后,又执行了安装任务 install:install 从输出可以看到该任务将项目输出的 jar 安装到了 Maven 本地仓库中,可以`打开相应的文件夹`看到本地pom和jar

- `C:\Users\xxName\.m2\repository\com\blogcode\helloworld\Hello_World\1.0-SNAPSHOT\Hello_World-1.0-SNAPSHOT.pom`
- `C:\Users\xxName\.m2\repository\com\blogcode\helloworld\Hello_World\1.0-SNAPSHOT\Hello_World-1.0-SNAPSHOT.jar`

> 之前讲述的 Junit 的 pom及jar 的下载的时候,我们说只有构建被下载到本地仓库后,才能由所有 maven 项目使用,这里是同样的道理,只有将Hello world 的构件安装到本地仓库之后,其他 maven 项目才能使用它

到目前为止,还没有运行HelloWorld项目,不要忘了HelloWorld类可是有一个main方法的。默认打包生成的jar是不能够直接运行的,因为带有`main方法的类信息`不会添加到manifest中（打开jar文件中的META-INF/MANIFEST.MF文件,将无法看到Main-Class一行）。为了`生成可执行的jar文件`,需要借助maven-shade-plugin,配置该插件如下：


```xml
<build>

    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <main-Class>com.blogcode.helloworld.HelloWorld</main-Class>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>

            </executions>
        </plugin>
    </plugins>
</build>
```

现在执行mvn clean install，待构建完成之后打开target/目录，可以看到hello-world-1.0-SNAPSHOT.jar和original-hello-world-1.0-SNAPSHOT.jar，前者是带有Main-Class信息的可运行jar，后者是原始的 jar


```shell
E:\Hello_World>java -jar target\Hello_World-1.0-SNAPSHOT.jar
Hello world!
```
> 注意:在生成包之后,运行包时发生了错误,`错误: 找不到或无法加载主类 com.blogcode.HelloWorld`,经过排查发现是手写类的`package`出现路径及拼写错误,估计手写就有这毛病....

# 使用 Archetype 生成项目骨架
前面都是手写项目骨架,Maven 提供了 Archetype 以帮助我们快速生成项目骨架

Maven 3
```shell
mvn archetype:generate
```
> Maven 3 中.即时没有指定版本,也会指定最新的稳定版本,因此这是安全的

工作流程
- 输入命令
- 默认回车选择的是 `maven-archetype-quickstart`
- 输入 groupid,artfactId,version,package



> archetype 插件将根据我们提供的信息创建项目骨架.其中
- groupid 是主代码的存放路径
- artfactId 是项目名称,所有代码存放在此文件夹
- version 开发版本
- package 这个一般根据 groupid 生成对应的,一般默认
- Y 确认生成


# Idea 下使用 Maven

- [eclipse 使用](https://blog.csdn.net/u012052268/article/details/78916196)
- idea 使用
