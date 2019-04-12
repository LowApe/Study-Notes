<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Maven 概述](#maven-概述)
- [Maven 的安装和配置](#maven-的安装和配置)
	- [安装目录分析](#安装目录分析)
	- [设置 http 代理](#设置-http-代理)
	- [安装 m2eclipse](#安装-m2eclipse)
	- [安装 NetBeans Maven 插件](#安装-netbeans-maven-插件)
- [Maven 安装最佳实践](#maven-安装最佳实践)
	- [设置 MAVEN_OPTS 环境变量:](#设置-mavenopts-环境变量)
	- [配置用户范围 settings.xml](#配置用户范围-settingsxml)
	- [不要使用 IDE 内嵌的 Maven](#不要使用-ide-内嵌的-maven)

<!-- /TOC -->
# Maven 概述
Maven作为一个构建工具,不仅能帮我们自动化构建,还能够抽象构建过程,提供构建任务实现;它跨平台,对外提供了一致的操作接口,这一切足以使它成为优秀的,流行的构建工具.

Maven 不仅是`构建工具`,还是一个`依赖管理工具`和`项目信息管理工具`.它提供了中央仓库,能帮我们自动下载构件

> 场景:java 应用都会借用一些第三方的开源类库,这些类库都可通过依赖的方式引入到项目中来.但随着依赖的增多,版本不一致,版本冲突,依赖臃肿的等问题,手工解决十分笨重

> Maven 通过坐标系统准确地定位每一个构建(artifact),也就是通过一组坐标能够找到任意一个Java类库.

> 场景:Maven 还能帮助我们管理原本分散在项目中各个角落的`项目信息`(项目描述,开发者列表,版本控制系统地址,许可证,缺陷管理系统地址)

> 除了直接项目信息,通过Maven自动生成的站点,以及一些已有的插件,我们还能获得项目文档,测试测试报告,静态分析报告,源码版本日志报告等项目信息

Maven 还提供免费的中央仓库,在其中几乎可以找到任何的流行开源类库.通过Maven衍生工具(Nexus),我们还能对其进行快速搜索

# Maven 的安装和配置
Maven 下载地址:[http://maven.apache.org/download.cgi](http://maven.apache.org/download.cgi)

## 安装目录分析
安装:
- 解压
- 环境变量配置 M2_HOME:路径;`path:%M2_HOME%\bin`;cmd下`mvn -v`检查

目录:
- bin (mvn运行脚本)
- boot (该目录只包含一个文件 类加载器框架 `jar` 文件)
- conf (`settings.xml` 全局定制 Maven 行为)
- lib (包含 Maven 运行是所需要的 Java 类库;用户可以在这个目录中找到 Maven 内置的超级 POM)
- LICENSE.txt (许可证)
- NOTICE.txt (Maven 包含的第三方软件)
- README.md

~/.m2:
- windows 一般在 C:\User\name\.m2\repository
- 大多数用户需要复制 `conf/settings.xml 到 ~/.m2/settings.xml`

## 设置 http 代理
有时候公司基于安全考虑,要求你使用安全认证代理访问英特网.这种就需要为 Maven 配置 http 代理,才能让他正常访问外部仓库,下载资源. `settings.xml` 中添加

## 安装 m2eclipse
Eclipse Maven插件:[http://m2eclipse.sonatype.org/](http://m2eclipse.sonatype.org/)

Eclipse 步骤:
- help
- install new software
- add
- 进入[https://www.eclipse.org/m2e/m2e-downloads.html](https://www.eclipse.org/m2e/m2e-downloads.html),选择url设置location:
- next 下载安装模块

组件用途
- 重要的
    - Maven SCM handler for Subclipse(Optional):Subversion是流行版本控制
    - Maven SCM Integration (Optional):Eclipse 环境中 Maven 与 SCM 集成核心模块
- 不重要的
    - Maven issue tracking configurator for Mylyn 3.x (Optional):该模块能够帮助我们使用 POM 中的缺陷跟踪系统信息连接 Mylyn致服务器
    - Maven SCM handler for Team/CVS(Optional):该模块帮助我们从 CVS 服务器签出 Maven 项目,如果还在使用  CVS,就需要安装它
    - Maven Integration for WTP (Optional):使用该模块可以让 Eclipse 自动读取 POM 信息配置 WTP 项目
    - M2Eclipse Extensions Development Support (Optional) 用来支持扩展 m2eclipse,一般用户不会用到
    - Project configuratiors for commonly used maven plugins(temporary):Maven 插件与 Eclipse 的集成,建议安装

安装后重启,file->new-> other,查看是否可以创建 maven

## 安装 NetBeans Maven 插件
flag:不懂

# Maven 安装最佳实践
## 设置 MAVEN_OPTS 环境变量:
运行 mvn 命令实际上是执行了 Java 命令,既然是运行 Java ,那么运行 Java 命令可用的参数当然也应该在运行 mvn 命令时可用.这个时候 MAVEN_OPTS 环境变量用法.

通常需要设置 MAVEN_OPTS 的值为 `-Xms128m-Xmx512m`,因为 Java 默认的最大可用内存往往不能够满足 Maven 运行的需要,比如在项目较大时,使用 Maven 生成项目站点需要占用大量内存,如果没有该配置,很容易得到 `java.lang.OutOfMemeoryError` 因此,一开始就配置该变量是推荐的做法.

关于环境变量,劲量不要直接修改mvn.bar 或者 mvn 这两个 Maven 执行脚本文件. 因为如果修改了脚本文件,升级 Maven 时就不得不修改,容易忘记

## 配置用户范围 settings.xml
可以在
- %M2_HOME%/conf/settings.xml
- ~/.m2/settings.xml

前者全局范围,后者用户

> 注意:推荐使用用户范围 settings.xml ,这也是为什么把全局的复制到用户设置,主要是为了避免无意识得影响到系统中的其他用户.如果有切实的需求,需要统一系统的设置,在全局进行设置.用用户还能方便用户升级

## 不要使用 IDE 内嵌的 Maven

Maven 无论 Eclipse 还是 NetBeans ,当集成 Maven 时,都会安装上一个内嵌的 Maven ,内嵌的比较新,但不是很稳定.这里存在两个潜在问题:
1. 新版本不稳定
2. 除了IDE,也经常还会使用`命令行`的Maven,如果版本不一致,会造成不同的构建行为.

因此在 IDE 中配置 Maven 插件时使用与命令行一致的 Maven
