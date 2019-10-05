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
- 或者通过 Source Bundle Releases 页面下载打包好的源码
