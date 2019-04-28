# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [何为 Maven 仓库](#何为-maven-仓库)
- [仓库的布局](#仓库的布局)
- [仓库的分类](#仓库的分类)
	- [本地仓库](#本地仓库)
	- [远程仓库](#远程仓库)
	- [中央仓库](#中央仓库)
	- [私服](#私服)
- [远程仓库的配置](#远程仓库的配置)
	- [远程仓库的认证](#远程仓库的认证)
	- [部署至远程仓库](#部署至远程仓库)
- [快照版本](#快照版本)
- [从仓库解析依赖的机制](#从仓库解析依赖的机制)
- [镜像](#镜像)
- [仓库搜索服务](#仓库搜索服务)

<!-- /TOC -->
# 何为 Maven 仓库
任何一个依赖,插件或者项目构建的输出都可以称为构件.

在一台工作站上，可能会有几十个 Maven 项目，所有项目都使用`maven-compiler-plugin`，这些项目中的大部分都用到了log4j，有一小部分用到了`SpringFramework`，还有另外一小部分用到了`Struts2`。在每个有需要的项目中都放置一份重复的log4j或者struts2显然不是最好的解决方案，这样做不仅造成了磁盘空间的浪费，而且也难于统一管理,文件的复制等操作也会降低构建的速度.而不使用 Maven 的哪些项目中,我们都放到 lib 目录下,这也会有大量重复.

Maven 可以在某个位置统一存储所有 Maven 项目共享的构建,这个统一的位置就是仓库. 实际的 Maven 项目将不再各自存储其依赖文件,它们只需要声明这些依赖的坐标,在需要的时候(例如:编译项目的时候需要将依赖加入到 classpath 中),Maven 会自动根据坐标找到仓库中的构建,并使用它们.

> 我们在电脑的项目,每次在 pom.xml 文件中声明依赖,并导入,其实文件并,没有复制到该项目,只是引入到了项目,直到打包才输出这些包.

# 仓库的布局
查看本地的 Maven 仓库(~/.m2/repository),分析结构,你会发现路径与坐标的大致对应关系 `groupId/artifactId/version/artifactId-version.packaging`

**Maven 处理仓库布局的源码**

```java
@Component( role = ArtifactRepositoryLayout.class, hint = "default" )
public class DefaultRepositoryLayout
    implements ArtifactRepositoryLayout
{
    private static final char PATH_SEPARATOR = '/';

    private static final char GROUP_SEPARATOR = '.';

    private static final char ARTIFACT_SEPARATOR = '-';

    public String getId()
    {
        return "default";
    }

    public String pathOf( Artifact artifact )
    {
        ArtifactHandler artifactHandler = artifact.getArtifactHandler();

        StringBuilder path = new StringBuilder( 128 );

        path.append( formatAsDirectory( artifact.getGroupId() ) ).append( PATH_SEPARATOR );
        path.append( artifact.getArtifactId() ).append( PATH_SEPARATOR );
        path.append( artifact.getBaseVersion() ).append( PATH_SEPARATOR );
        path.append( artifact.getArtifactId() ).append( ARTIFACT_SEPARATOR ).append( artifact.getVersion() );

        if ( artifact.hasClassifier() )
        {
            path.append( ARTIFACT_SEPARATOR ).append( artifact.getClassifier() );
        }

        if ( artifactHandler.getExtension() != null && artifactHandler.getExtension().length() > 0 )
        {
            path.append( GROUP_SEPARATOR ).append( artifactHandler.getExtension() );
        }

        return path.toString();
    }

    public String pathOfLocalRepositoryMetadata( ArtifactMetadata metadata, ArtifactRepository repository )
    {
        return pathOfRepositoryMetadata( metadata, metadata.getLocalFilename( repository ) );
    }

    private String pathOfRepositoryMetadata( ArtifactMetadata metadata,
                                             String filename )
    {
        StringBuilder path = new StringBuilder( 128 );

        path.append( formatAsDirectory( metadata.getGroupId() ) ).append( PATH_SEPARATOR );
        if ( !metadata.storedInGroupDirectory() )
        {
            path.append( metadata.getArtifactId() ).append( PATH_SEPARATOR );

            if ( metadata.storedInArtifactVersionDirectory() )
            {
                path.append( metadata.getBaseVersion() ).append( PATH_SEPARATOR );
            }
        }

        path.append( filename );

        return path.toString();
    }


}

```
该 pathOf() 方法的目的是根据构件信息生成其在仓库中的路径.考虑这样一个构建: groupId= org.testng,artifact=testng,version=5.8,classifier=jdk15,packaging=jar,然后根据源码的路径如下步骤:

1. 基于构建的 groupId 准备路径, formatAsDirectory() 将 groupId 中的句点分隔符转换成路径分隔符.例如:groupId=org.testing 被转换成 org/testing
    ```java
    private String formatAsDirectory( String directory )
       {
           return directory.replace( GROUP_SEPARATOR, PATH_SEPARATOR );
       }    
    ```
- 基于构建的 artifactiId 准备路径,也就是前面的基础上加上 artifactId 准备路径,也就是前面的基础上加上 artifactId 以及一个路径分隔符.改例中的 artifactId 为 testing ,那么这一步过后,路径就成为了 org/testng/testng/
    ```java
    path.append( artifact.getArtifactId() ).append( PATH_SEPARATOR );    
    ```
- 使用版本信息.在前面的基础上加上 version 和路径分隔符/例如版本 5.8.那么路径就成为了 org/testng/testng/5.8
    ```java
    path.append( artifact.getBaseVersion() ).append( PATH_SEPARATOR );
    ```
- 依次加上 artfactId,构建分隔符连字号,以及 version.于是构建的路径就变成 org/testng/testng/5.8/testnb-5.8;这里使用了 getVersion() ,前面使用的是 getBaseVersion(),因为后者主要是 SNAPSHOT 的构件,其 baseVersion 就是 1.0.
    ```java
    path.append( artifact.getArtifactId() ).append( ARTIFACT_SEPARATOR ).append( artifact.getVersion() );
    ```
- 如果构件有 classifier,就加上构件分隔符和 classifier.该例中构件的 classifier 是 jdk15,那么路径就变成 org/testng/testng/5.8/testng-5.8-jdk5
    ```java
    if ( artifact.hasClassifier() )
         {
             path.append( ARTIFACT_SEPARATOR ).append( artifact.getClassifier() );
         }    
    ```
- 检查构件的 extension,若 extension 存在,则加上句点分隔符和 extension.从代码看出,extension 是从 artifactHandler 而非 artifact 获取,artifactHandler 是由项目的 packaging 决定的
    ```java
    if ( artifactHandler.getExtension() != null && artifactHandler.getExtension().length() > 0 )
       {
           path.append( GROUP_SEPARATOR ).append( artifactHandler.getExtension() );
       }
    ```
Maven仓库是基于简单文件系统存储的，我们也理解了其存储方式，因此，当遇到一些与仓库相关的问题时，可以很方便地查找相关文件，方便定位问题。例如，当Maven无法获得项目声明的依赖时，可以查看该依赖对应的文件在仓库中是否存在，如果不存在，查看是否有其他版本可用，等等

# 仓库的分类

对于 Maven 来说,仓库只分为两类,本地仓库和远程仓库.当 Maven 根据坐标寻找构件,先查看本地仓库,如果没有查看是否有更新构件版本,Maven就去远程仓库查找,下载到本地使用,如果本地和远程都没有则会报错.

还有特殊的仓库,私服:为了节省带宽和时间,应该在局域网内架设一个私有的仓库服务器,用其代理所有外部的远程仓库.

**Maven 仓库分类**

![](http://ww1.sinaimg.cn/large/006rAlqhly1g27xoytlu7j30bd05cgm0.jpg)

除了中央仓库和私服,还有很多其他公开的远程仓库,常见的有 [Java.net Maven 库]()和 [JBoss Maven 库](http://repository.jboss.com/maven2/)

## 本地仓库
默认情况下,不管是在 Windows 还是 Linux 上,每个用户在自己的用户目录下都有一个路径名 `.m2/repository` (c://users/电脑名称/.m2/repository)的仓库目录(Linux ls -a 显示隐藏的 .).

> 有时候,需要自定义本地仓库目录地址(C盘不足),编辑文件 ~/.m2/settings.xml,设置 localRepository 元素的值为想要的仓库地址

```xml
<settings>
    <localRepository>E:/MavenRepository/repository</localRepository>
</settings>
```

## 远程仓库
新安装的 Maven,不执行命令,本地仓库是不存在的.当用户输入第一条 Maven,从远程仓库下载构建到本地仓库.

## 中央仓库
由于最原始的本地仓库是空的，Maven必须知道至少一个可用的远程仓库，才能在执行Maven命令的时候下载到需要的构件。中央仓库就是这样一个默认的远程仓库，Maven的安装文件自带了中央仓库的配置。使用解压工具打开jar文件`$M2_HOME/lib/maven-model-builder-3.0.jar`（在Maven2中，jar文件路径类似于`$M2_HOME/lib/maven-2.2.1-uber.jar`），然后访问路径org/apache/maven/model/pom-4.0.0.xml，可以看到如下的配置:
```xml
<repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
</repository>
```
这段配置 id central 对中央仓库进行唯一标识,最后注意是 snapshots 元素,子元素 false,表示不从该中央仓库下载快照版本的构件.

## 私服
私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的Maven用户使用。当Maven需要下载构件的时候，它从私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，缓存在私服上之后，再为Maven的下载请求提供服务。此外，一些无法从外部仓库下载到的构件也能从本地上传到私服上供大家使用

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g27yf7dn12j30gy0dntbi.jpg)

私服可以:
- 节省自己的外网带宽
- 加速 Maven 构建
- 部署第三方构件
- 提高稳定性,增强控制
- 降低中央仓库的负荷

后面介绍使用最流行的 Maven 私服软件 Nexus;

# 远程仓库的配置
可能项目需要的构件存在于另外一个远程仓库中,如 JBoss Maven 仓库. 这时,可以在 POM(超级 POM) 中配置该仓库

```xml
<repository>
      <id>jboss</id>
      <name>JBoss Repository</name>
      <url>https://repository.jbos.com/maven2</url>
      <releases>
          <enabled>true</enabled>
      </release>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
</repository>

```
> layout 元素值 default 表示仓库的布局是 Maven 2 和 3 的默认布局;对于 releases 和 snapshots 来说,除了 enabled,它们还包含另外两个子元素 updatePolicy 和 checkssumPlicy

```xml
<snapshots>
  <enabled>false</enabled>
  <updatePolicy>daily</updatePolicy>
  <checkssumPlicy>ignore</checkssumPlicy>
</snapshots>
```

updatePolicy 从远程仓库检查的更新的频率,默认的值是 daily,表示 Maven 每天检查一次(有 never always interval:X-几分钟 等属性).  checkssumPlicy 用来配置 Maven 检查检验和文件的策略.

## 远程仓库的认证

大部分远程仓库无须认证就可以访问，但有时候出于安全方面的考虑，我们需要提供认证信息才能访问一些远程仓库。例如，组织内部有一个Maven仓库服务器，该服务器为每个项目都提供独立的Maven仓库，为了防止非法的仓库访问，管理员为每个仓库提供了一组用户名及密码。这时，为了能让Maven访问仓库内容，就需要配置认证信息。


配置`认证信息`和配置`仓库信息`不同，仓库信息可以直接配置在POM文件中，但是认证信息必须配置在`settings.xml`文件中。这是因为 POM 往往是被提交到代码仓库中供所有成员访问的，而 `settings.xml`一般只放在本机。因此，在`settings.xml`中配置认证信息更为安全。假设为 my-project 构件添加认证信息

```xml
<settings>
    ...
    <server>
      <id>my-project</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    ...
</settings>    
```

> 这个 id 将认证信息与仓库配置联系在一起

## 部署至远程仓库
前面提到,私服的一大作用是部署第三方构件,包括组织内部生成的构件以及一些无法从外部仓库直接获取的构件.

Maven 除了能对项目进行编译,测试,打包之外,还能将项目生成的构建部署到仓库中.首先,需要编辑项目的 `pom.xml` 文件. 配置 distributionManagement 元素

```xml
<distributionManagement>
	<repository>
		<id>pro-release</id>
		<name>Proj Release Repository</name>
		<url>http://localhost:8081/nexus/content/repositories/RestBus-Releases</url>
	</repository>

	<snapshotRepository>
		<id>pro-snapshot</id>
		<name>Proj Snapshot Repository</name>
		<url>http://localhost:8081/nexus/content/repositories/RestBus-Snapshots</url>
	</snapshotRepository>
</distributionManagement>
```
distributionManagement 包含 repository 和 snapshotRepositrory 子元素,前者表示发布版本构件的仓库,后者表示快照版本的仓库;两个都需要配置 id,name 和 url,id为该远程仓库唯一标识,name 是为了方便人阅读,关键的 url 表示该仓库的地址,还有一般上传都要认证,所以要在 settings.xml 中创建一个 server 元素.


# 快照版本
2.1-SNAPSHOT 和 2.1-20091214.221414-13 是不稳定版本的快照版本

Maven 为什么要区别分发布版好快照版?
> 场景: 甲开发模块 A 的2.1版本,该版本未正式发布,与模块 A 一同开发还有模块 B ,它由甲的同事乙开发, B的功能依赖于 A. 在开发过程中甲经常将自己最新的构建输出,交给乙,问题是,工作如何进行?

1. 乙签出模块 A 的源码进行构建.乙不得不去构建模块 A.多了版本控制和 Maven 操作还不算,当构建 A 失败的是时候,最后不得不找甲.这种方式是低效的

- 重复部署模块2.1版本供乙使用,对于Maven来说,同样的版本和坐标意味着同样的构建.因此,如果乙在本机的本地仓库包含了 模块A 的2.1版本,`Maven就不会在对照远程仓库进行更新`.除非每次清除本地.
- 不停更新版本号,这是对版本号的滥用.


Maven 的快照版本机制就是为了解决上诉问题.将模块 A的 版本设定为 2.1-SNAPSHOT,然后发布到私服中,在发布过程中, Maven 会自动为构件打上时间戳.比如 2.1-20191214.221414-13就表示2019年12月14日22点14分14秒的第13次快照.有了时间戳, Maven 就能随时找到仓库中该构件 2.1-SNAPSHOT 版本的最新文件.当乙构件 B 的时候,Maven 会自动从仓库中检查模块 A 的 2.1-SNAPSHOT 的最新构件,有最新便进行下载. `mvn clean install -U` -U强制更新

# 从仓库解析依赖的机制

Maven 是根据怎样的规则从仓库解析并使用依赖构件的呢?
1. 当依赖的范围是 system 的时候,Maven 直接从本地文件系统解析构件.
2. 根据`依赖坐标计算`仓库路径后,尝试直接从本地仓库寻找构件,如果发现相应则解析成功
3. 在本地仓库不存在相应构建,如果依赖的版本是显式的发布版本构件,如1.2,2.1-beta-1等,则遍历所以的远程仓库,发现后,下载并解析使用.

4. 如果依赖的版本是 REALEASE 或者 LATEST,则基于更新策略读取所有远程仓库的元数据 `groupId/artifactId/maven-metadata.xml` 将其与本地仓库的对应元数据合并后,计算出 RELEASE 或者 LATEST 真实的值,基于这个值检查本地和远程仓库,如步骤2,3;
5. 如果依赖的版本是SNAPSHOT，则基于更新策略读取所有远程仓库的元数据`groupId/artifactId/version/maven-metadata.xml`，将其与本地仓库的对应元数据合并后，得到最新快照版本的值，然后基于该值检查本地仓库，或者从远程仓库下载。

6. 如果最后解析得到构件版本是时间戳式的快照,如1.4.1-20191104.121450-121,则复制其时间戳格式的文件至非时间戳格式,如 SNAPSHOT ,并使用非时间戳格式的构件.


当依赖的版本不明晰的时候，如RELEASE、LATEST和SNAPSHOT，Maven就需要基于更新远程仓库的更新策略来检查更新。在仓库配置中，有一些配置与此有关：首先是`<releases><enabled>`和`<snapshots><enabled>`，只有仓库开启了对于发布版本的支持时，才能访问该仓库的`发布版本构件信息`，对于快照版本也是同理；其次要注意的是`<releases>`和`<snapshots>的子元素<updatePolicy>`，该元素配置了检查更新的频率，每日检查更新、永远检查更新、从不检查更新、自定义时间间隔检查更新等。最后，用户还可以从命令行加入参数-U，强制检查更新，使用参数后，Maven就会忽略`<updatePolicy>` 的配置.

```xml
<versioning>
  <versions>
    <version>1.0-SNAPSHOT</version>
  </versions>
  <lastUpdated>20190419075853</lastUpdated>
</versioning>
```
> Maven3 不在支持在插件配置中使用 LATEST 和 RELEASE.

> flag:这块因为没有时间开发,所有这个文件信息并不全,不是能很好理解,以后理解吧...

# 镜像
比如,国内使用阿里云镜像.

> 需要注意的是，由于镜像仓库完全屏蔽了被镜像仓库，当镜像仓库不稳定或者停止服务的时候，Maven仍将无法访问被镜像仓库，因而将无法下载构件


# 仓库搜索服务
使用 Maven 进行日常开发的时候,如何寻找需要的依赖,我们可能知道类库项目名称,但添加确切 maven 坐标,通过`仓库搜索服务`来根据关键字得到 Maven 坐标

- Sonatype Nexus  [https://repository.sonatype.org/](https://repository.sonatype.org/)
- maven [http://repo1.maven.org/](http://repo1.maven.org/)
- MVNrepository [https://mvnrepository.com/](https://mvnrepository.com/)
