# 概述
Java 不仅是一门编程语言，还是一系列软件与规范形成的**技术体系**。这个技术体系提供了完整的用于**软件开发和跨平台部署的支持环境**，并广泛应用于各种场合

Java 广泛的认可：
- 是一门结构严谨、**面向对象的** 编程语言
- 跨平台--一次编写、到处运行
- 提供相对安全的 **内存管理和访问机制**
- 实现 **热点代码检测和运行时编译及优化** 提高Java的性能
- 完善的 **应用程序接口**

# Java技术体系

Sun 官方所定义的Java技术体系：
- Java 程序设计语言
- 各种硬件平台的Java虚拟机
- Class 文件格式
- Java API 类库
- 商业机构和开源社区的第三方类库

> 把Java程序设计语言、Java虚拟机、Java API类库称为**JDK(Java Development Kit)** JDK 是用于支持 Java 程序开发的最小环境。<br>
把Java API 类库的**Java SE API**子集和**Java虚拟机**着两部分称为**JRE(Java Runtime Envvironment)** JRE是支持Java程序运行的标准环境

按照 Java 技术关注的重点**业务领域**，Java 技术平台可以分为4个平台
- Java Card
- Java ME
- Java SE
- Java EE

# Java的发展史

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7n4zj76nvj324m0s07ii.jpg)

> 最新请查看--[Java发展史](https://github.com/didiaoyuan/KGGO/tree/master/Java)

# Java 虚拟机发展史
内容比较多，以后详细归纳

# 展望Java技术的未来
- 模块化
- 混合语言
- 多核并行
    - jdk1.7 java.util.concurrent.forkjoin 利用多核心提供计算资源
- 进一步丰富语法
- 64位虚拟机

# 实战：自己编译 JDK

## 1.获取JDK

- 通过 Mercurial 代码版本管理工具
- 或者通过 Source Bundle Releases
- [oepnjdk](http://hg.openjdk.java.net/jdk-updates)
## 2.构建编译系统
MacOs准备
- XCode()
- Command Line Tools for XCode
- 6u14以上版本的JDK(**jdk版本尽量低于openJDK版本，否则坑太多**)
- Apache Ant


> ⚠️注意:还有不同的openJDK 对应不同JDK版本，比如1.8不能编译JDK13，最少JDK11、12版本的JDK(Bootstrap JDK，不是OpenJDK)


## 3. 进行编译
需要下载的**编译环境和依赖项目**都准备齐全了，最后需要对系统的环境变量做一些简单设置以便编译能够顺利通过

OpenJDK 在编译时读取的环境变量比较多，但都有默认值，必须设置的有两个
- LANG 设置语言选项
    ```
    设置为 export LANG = C ,否则，在编译结束前的**验证阶段**出现 HashTable 内的空指针异常
    ```
- ALT_BOOTDIR 配置 Booststrap JDK 路径

    ~~export ALT_BOOTDIR=/Library/Java/JavaVirtualMachine/openjdk-13.jdk/Contents/Home~~


> ⚠️注意：如果读者之前设置了 JAVA_HOME 和 CLASSPATH，在编译之前必须取消，否则在 Makefile 脚本中检查到有这两个变量存在，会有警告提示
```
unset JAVA_HOME
unset CLASSPATH
```

****

> 按照书上的好些都不管用，而且网上讲的也不太清楚，一堆工具，还有一些步骤，最后我找到一个成功了

成功环境
- jdk-13-33
- 编译openJDK13-33
> 编译完成后，并没有编译信息，有一些错误，但是可以通过`./build/macosx-x86_64-server-slowdebug/jdk/bin/java --version`验证了。

- [成功编译openJDK13-33](https://www.jianshu.com/p/37101a4a35f9?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g7oroligksj30uw06mjyh.jpg)

## 在 IDE 工具中进行源码调试

- OS：Mac系统
- IDE：Clion
- 代码：OpenJDK10

[使用Clion查看源码](https://blog.csdn.net/wd2014610/article/details/81703203)

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1g7osvdzlo8j31ms0xohdt.jpg)

# 构建过程中遇到的问题    
为了测试是否`jdk版本尽量低于openJDK版本`,我又下载了
- JDK8u
- 以及OpenJDK8u
- OpenJDK9
进行测试，其中会用到[切换jdk版本](https://blog.csdn.net/u012198209/article/details/79100727)
出现下面的问题,后来还是没解决，因为底层真的不懂，暂时先用前面编译好的13版本的openJDK😢
## error 1 gcc compiler is required

```shell
checking for jtreg... no
checking for clang... /usr/bin/clang
configure: Resolving CC (as /usr/bin/clang) failed, using /usr/bin/clang directly.
checking resolved symbolic links for CC... /usr/bin/clang
checking if CC is disguised ccache... no, keeping CC
configure: The C compiler (located as /usr/bin/clang) does not seem to be the required GCC compiler.
configure: The result from running with --version was: "Apple LLVM version 10.0.0 (clang-1000.11.45.2)"
configure: error: GCC compiler is required. Try setting --with-tools-dir.
configure exiting with result code 1
```
解决方法：YourOpenJDK/common/autoconf/generated-configure.sh里边校验抛出，我直接找到下边文本出现的两个地方给注释掉了......

```
as_fn_error $? "GCC compiler is required. Try setting --with-tools-dir." "$LINENO" 5
```

## error 2 could not find freetype

在yourOpenJDK 下 `configure` 命令

```shell
bash ./configure --with-debug-level=slowdebug --with-boot-jdk=/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home
```

出现 not find freetype 错误，通过安装`XQuartz`来安装freetype,在上面命令基础上添加下面的命令

```
--with-freetype-include=/opt/X11/include/freetype2/ --with-freetype-lib=/opt/X11/lib
```
