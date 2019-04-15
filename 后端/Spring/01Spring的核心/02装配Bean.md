# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [Spring 配合的可选方案](#spring-配合的可选方案)
- [自动化装配 bean](#自动化装配-bean)
	- [创建可被发现的 bean](#创建可被发现的-bean)
	- [为组件扫描的 bean 命名](#为组件扫描的-bean-命名)

<!-- /TOC -->

# Spring 配合的可选方案
创建应用对象之间协作关系的行为通常称为装配(wiring),这也是依赖注入(DI)本质.
Spring 容器负责创建应用程序的 bean 并通过 DI 来协调这些对象之间的关系.

但是,作为开发人员,你需要告诉 Spring 要创建哪些 bean 并且如何其装配(bean 与 创建的实体)在一起.当描述 bean 如何进行装配时,Spring 具有非常大的灵活性,它提供类三种主要的装配机制:
- 在 XML 中进行显式配置
- 在 Java 中进行显式配置
- 隐式的 bean 发现机制和自动装配

> 建议是尽可能地使用自动装配的机制.显式配置越少越好.当你必须要显式配置 bean 的时候,推荐 JavaConfig.如果想使用遍历的 XML 命名空间,并且在 JavaConfig 中没有同样的实现时,才应该使用 XML

# 自动化装配 bean

Spring 从两个角度来实现自动化装配:
- 组件扫描(component scanning):Spring 会自动发现应用上下文中所创建的 bean
- 自动装配(autowiring):Spring 自动满足 bean 之间的依赖.

## 创建可被发现的 bean

```java
public interface CompactDisc {
    void play();
}

// --分割线--

import org.springframework.stereotype.Component;
import top.blogcode.imp.CompactDisc;

@Component
public class SgtPepers implements CompactDisc {
    private String title = "Sgt. Pepper's Lonely Hearts Club Band";
    private String artist = "The Beatles";

    public void play() {
        System.out.println("Playing " + title + " by " + artist);
    }
}

```
这个`@Component`注解表明该类会作为组件类,并告知 Spring 要为这个类创建 bean.所有不用显示配置

不过,`组件扫描`默认是不启动的.我们还需要显式配置一下 Spring

```java

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan // @ComponentScan 注解启动组件扫描
public class CDPlayerConfig {
}

```
如果没有其他配置的话,`@ComponentScan` 默认会扫描与配置类相同的包.当发现 `@Component` 注解的类就会自动创建一个 bean

**通过XML启动组件扫描**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

  <context:component-scan base-package="top.blogcode.pojo" />

</beans>
```

写一个 CDPlayerTest类进行测试

```java
import static org.junit.Assert.*;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;


@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=CDPlayerConfig.class)
public class CDPlayerConfigTest {

    @Autowired
      private CompactDisc cd;

      @Test
      public void testCdShouldNotBeNull(){
          assertNotNull(cd);
      }
}
```

CDPlayerConfig 使用了 Spring-test 的 SpringJUnit4ClassRunner,以便在测试开始的时候自动创建 Spring `的应用上下文`.

> 注意: `@ContextConfiguration`会告诉它需要在 CDPlayerConfig 中加载配置.因为 CDPlayerConfig 类中包含 `@ComponentScan`,因此最终的`应用上下文`中应该包含 CompactDisc bean.

测试代码中 CompactDisc 属性带有 `@Autowired` 注解,以便于将 CompactDisc bean 注入到测试代码之中

## 为组件扫描的 bean 命名
