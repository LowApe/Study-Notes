# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [Spring 配合的可选方案](#spring-配合的可选方案)
- [自动化装配 bean](#自动化装配-bean)
	- [创建可被发现的 bean](#创建可被发现的-bean)
	- [为组件扫描的 bean 命名](#为组件扫描的-bean-命名)
	- [设置组件扫描的基础包](#设置组件扫描的基础包)
	- [通过为 bean 添加注释实现自动装配](#通过为-bean-添加注释实现自动装配)
	- [验证自动装配](#验证自动装配)
- [通过 Java 代码装配 bean](#通过-java-代码装配-bean)
	- [创建配置类](#创建配置类)
	- [声明简单的 bean](#声明简单的-bean)
	- [借助 JavaConfig 实现注入](#借助-javaconfig-实现注入)
- [通过 XML 装配 bean](#通过-xml-装配-bean)
	- [创建 XML 配置规范](#创建-xml-配置规范)
	- [声明一个简单的 bean](#声明一个简单的-bean)
	- [借助构造器注入初始化 bean](#借助构造器注入初始化-bean)
		- [构造器注入 bean 引用](#构造器注入-bean-引用)
		- [将字面量注入到构造器中](#将字面量注入到构造器中)
		- [装配集合](#装配集合)
	- [设置属性](#设置属性)
		- [将字面量注入到属性中](#将字面量注入到属性中)
- [导入和混合配置](#导入和混合配置)
	- [在 JavaConfig 中引用 XML 配置](#在-javaconfig-中引用-xml-配置)
	- [在 XML 配置中引用 JavaConfig](#在-xml-配置中引用-javaconfig)

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
Spring 应用上下文中所有的 bean 都会给定一个 ID.具体来讲,这个 bean ID 就是将类名的第一个字母变为小写.

如果想要为这个 bean 设置不同的 ID,你将期望的 ID 作为值传递给 `@Component` 注解,比如:

```java
@Component("newName")
public class SgtPeppers implements CompactDisc{
	...
}
```

还有另外一种为 bean 命名的方式,使用 Java 依赖注入规范中所提供的 `@Named` 注释来为 bean 设置

```java
@Named("newName")
public class SgtPeppers implements CompactDisc{
	...
}
```

## 设置组件扫描的基础包
`@ComponentScan` 设置任何属性.默认`配置类`所在的包作为基础包来扫描组件.但如果扫描不同包怎么办?

如果将`配置类` 放在单独的包,默认就失效了,如何启动组件扫描?
1. `@ComponentScan("name")` 注解的 value 属性中指定包的名称
- `@ComponentScan(basePackages={"one","two"})` 设置多个基础包,利用数组(String 类型不安全)
- `@ComponentScan(basePackageClasses={one.class,two.class})` 其中是组建类,

## 通过为 bean 添加注释实现自动装配
自动装配就是让 Spring 自动满足 bean 依赖的一种方式,在满足依赖的过程中,会在 Spring 应用上下文中寻找匹配某个 bean 需求的其他  bean. 为了声明要进行自动装配,我们可以借助 Spring 的 `Autowiring` 注解

```java
package soundsystem;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CDPlayer implements MediaPlayer {
  private CompactDisc cd;

  @Autowired
  public CDPlayer(CompactDisc cd) {
    this.cd = cd;
  }

  public void play() {
    cd.play();
  }

}
```
> 上面代码,它的构造器上添加了 `@Autowired` 注解,这表明当 Spring 创建 CDPlayer bean 的时候,会通过这个构造器来实例化并且会传入一个可设置给 `CompactDisc `类型的 bean,构造器上参数的自动装配


```java
@Autowired
public void setCompactDisc(CompactDisc cd) {
  this.cd = cd;
}
```

> `@Autowired` 注解不仅能够用在构造器上,还能用在属性的 Setter 方法上,满足 `参数`上面的声明的依赖.为了`避免异常`你可以讲 `@Autowired` 的 required 属性设置为 false:
```java
@Autowired(required=false)
public CDPlayer(CompactDisc cd) {
  this.cd = cd;
}
```

> required 为 false ,Spring 先执行自动装配,没有匹配的 bean 的话,Spring 将会让这个 bean 处于未装配状态,不会报错.自动装配的歧义性:有多个 bean 都能满足依赖关系的话, Spring 将会抛出一个异常,表明没有明确的指定要选择哪个 bean 进行自动装配

`@Autowired` 是 Spring 特有的注解.`@Inject` 注解来源于 Java 依赖注入规范.

## 验证自动装配
> flag 第四版 Spring Action 需要导入好多依赖,估计书很久了.
# 通过 Java 代码装配 bean
如果自动化配置的方案行不通,因此需要明确配置 Spring.比如,你想讲第三方组件中的组件装配到你的应用中,在这种情况下,是没有办法在它的类上添加注解的.

进行显式配置时,JavaConfig 更为强大,类型安全并且对重构友好.因为它就是 Java 代码.JavaConfig 与其他的 Java 代码有所区别,在概念上,它与应用程序中的`业务逻辑`和`领域代码`是不同的.意味着 JavaConfig 不应该侵入到业务逻辑代码中

## 创建配置类

创建 JavaConfig 类的关键在于为其添加 `@Configuration` 注解,它表明这个类是一个配置类,该类应该包含在 Spring 应用上下文中如何创建 bean 的细节.

## 声明简单的 bean

要在 JavaConifg 中声明 bean,我们需要编写一个方法,这个方法会创建所需类型的实例,然后给这个方法添加 `@Bean`注解

```java
@Configuration
public class CDPlayerConfig {
    @Bean
    public CompactDisc sgetppers() {
        return new SgtPeppers();
    }
}
```

`@Bean` 注解会告诉 Spring 这个方法将会返回一个对象,该对象要注册为 Spring 应用上下文中的 bean.方法体中包含了最终生产 bean 实例的逻辑.

> bean 的名字一般小写,如果修改可以使用 `@Bean` 注解的name 属性

```java
@Bean(name="newName")
public CompactDisc sgetppers() {
	return new SgtPeppers();
}
```


## 借助 JavaConfig 实现注入
我们前面所声明的 CompactDisc bean 是非常简单的，它自身没有其他的依赖。但现在，我们需要声明 CDPlayer bean，下面代码表示它依赖于 CompactDisc。
```java
public class CDPlayer {
    private CompactDisc cd;
    @Autowired(required=false)
    public CDPlayer(CompactDisc cd) {
        this.cd = cd;
    }
}
```

在JavaConfig中，要如何将它们装配在一起呢？在JavaConfig中装配 bean 的最简单方式就是`引用创建` bean 的方法。例如，下面就是一种声明CDPlayer的可行方案：

```java
@Configuration
public class CDPlayerConfig {
    @Bean
    public CompactDisc sgetppers() {
        return new SgtPeppers();
    }
    @Bean
    public CDPlayer cdPlayer(){
        return new CDPlayer(sgetppers());
    }
}
```
cdPlayer() 方法使用 `@Bean` 注解表明这个方法会创建一个 bean 实例并将其注册到 Spring 应用上下文中.

默认情况下, Spring 中的 bean 都是单例的,假如对 sgtpeppers() 的调用就像 Java 方法调用一样,那么每个 CDPlayer 实例都会有一个特定的实例(其实不是,bean 是单例的),Spring 会拦截对 sgtpeppers() 的调用并确保返回的是 Spring 所创建的 bean(Sgtpeppers 那个 bean),因此下面的两个CDPlayer 会得到相同的 SgtPeppers 实例

```java
@Bean
public CDPlayer cdPlayer() {
  return new CDPlayer(sgtPeppers());
}

@Bean
public CDPlayer anotherCDPlayer() {
  return new CDPlayer(sgtPeppers());
}
```

利用调用方法引用 bean 方式有点困惑.其实还有一种理解起来更为简单的方式:

```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc) {
  return new CDPlayer(compactDisc);
}
```

在这里, cdPlayer() 方法请求一个 CompactDisc 参数,当 Spring 调用这个方法创建该类的 bean 时,它会自动装配一个 CompactDisc 到配置方法之中.借助这种技术,cdPlayer() 方法也能够将 CompactDisc 注入到 CDPlayer 的构造其中,而且不用明确引用 CompactDisc 的 `@Bean` 方法

- 这种方式引用其他的 bean 通常时最佳选择.因为它不要求将 CompactDisc 声明到同一个配置类中.
- 甚至没有要求 CompactDisc 必须要在 JavaConfig 中声明,实际上它可以通过组件扫米功能自动发现或者通过 XML 进行配置

- 另外这里我们构造器实现了 DI 功能,但是我们完全可以采用其他风格的 DI 配置(构造或者setter)

# 通过 XML 装配 bean
目前用的多的 Java 装配和自动化装配,因为 XML 方式流行很多年,学懂 XML 对修改以前代码有很大帮助

## 创建 XML 配置规范
创建一个 XML 文件,并且要以`<beans>` 元素为根

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

> 可以使用 idea 自动生成一个 `springconfig` 模版,作为起步,JavaConfig 中只需要 `@Configuration`,但在使用 XML 时,需要在配置文件的顶部声明多个 XML 模式(XSD)文件,

**借助 Spring Tool Suite 创建 XML  配置文件** 创建和管理 Spring XML 配置文件的一种简便方式


## 声明一个简单的 bean

```xml
<bean class="top.blogcode.pojo.SgtPeppers"/>
```

因没有明确给定 ID ,所有这个 bean 将会根据全限定类名来进行命名.在上面这个例子 bean 的 ID 将会"pojo.SgtPepper#0"
"#0"是一个计数形式.

## 借助构造器注入初始化 bean

在 Spring XML 配置中,只有一种声明 bean 的方式: 使用 <bean> 元素并指定 class 属性.Spring 会从这里获取必定的信息来创建 bean .

但是,在 XML 中声明 DI 时,会有多种可选的配置方案和分格.具体到构造器注入,有两种基本配置方案可选
- `<constructor-arg>`元素
- 使用 Spring 3.0 所引入的 c-命名空间

`<constructor-arg>` 方法,进行注入

### 构造器注入 bean 引用

```xml
<bean id="sgtpeppers" class="top.blogcode.pojo.SgtPeppers"/>
    <bean id="cdplayer" class="top.blogcode.pojo.CDPlayer">
        <constructor-arg ref="sgtpeppers"/>
    </bean>
```

**使用 Spring 的c-命名空间**

```xml
<!-- 第一步 -->
xmlns:c="http://www.springframework.org/schema/c"

<!-- 第二步 -->
<bean id="cdplayer" class="top.blogcode.pojo.CDPlayer" c:cd-ref="sgtpeppers"/>
```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g24gjw1edij308z050t94.jpg)
> 属性名以 "c:" 开头,也就是命名空间的前缀.接下来就是要`装配`的`构造器参数名`
,在此后是"-ref",这个命名约定,它会告诉 Spring , 正在装配的是一个 bean 的引用,这个 bean 的名字是 compactDisc .我们通过将参数名称替换成"0",也就是参数的索引.因为在 XML 中不允许数字作为属性的第一个字符,因此必须要添加一个下划线作为前缀

```xml
 <bean id="cdplayer" class="top.blogcode.pojo.CDPlayer" c:_0-ref="sgtpeppers"/>
```
### 将字面量注入到构造器中

迄今为止,我们所做的 DI 通常指的都是类型的装配(可以理解就是形参),而有时候,我们需要做的只是用一个字面量值来配置对象.

```java

public class BlankDisc implements CompactDisc {

  private String title;
  private String artist;

  public BlankDisc(String title, String artist) {
    this.title = title;
    this.artist = artist;
  }

  public void play() {
    System.out.println("Playing " + title + " by " + artist);
  }
}
```

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
</bean>
```
ref 修改成 value 就能把字面量值装配到 bean 中如果使用 c-命名空间的话,这个例子如下:
- 引用构造器参数名称
	```xml
	<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      c:_title="Sgt. Pepper's Lonely Hearts Club Band"
      c:_artist="The Beatles" />
	```
- 参数索引装配相同
	```xml
	<bean id="compactDisc"
	      class="soundsystem.BlankDisc"
	      c:_0="Sgt. Pepper's Lonely Hearts Club Band"
	      c:_1="The Beatles" />
	```
- 如果只有一个构造参数
	```xml
	<bean id="compactDisc" class="soundsystem.BlankDisc"
      c:_="Sgt. Pepper's Lonely Hearts Club Band" />
	```
### 装配集合
如果给我们的实体类添加一个属性,构造器参数也增加一个集合类该怎么办,集合类的配置 bean 产生影响,最简单的方式是将列表设置为 null

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
  <constructor-arg><null/></constructor-arg>
</bean>
```

但在注入期它能正常执行.当调用 play() 方法时,你会遇到 NullPointerException 异常,因此这并不是理想的方案.

- 可将 `<list>` 元素将其声明为一个列表
	```xml
	<bean id="compactDisc" class="soundsystem.BlankDisc">
	  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
	  <constructor-arg value="The Beatles" />
	  <constructor-arg>
	    <list>
	      <value>Sgt. Pepper's Lonely Hearts Club Band</value>
	      <value>With a Little Help from My Friends</value>
	      <value>Lucy in the Sky with Diamonds</value>
	      <value>Getting Better</value>
	      <value>Fixing a Hole</value>
	      <!-- ...other tracks omitted for brevity... -->
	    </list>
	  </constructor-arg>
	</bean>
	```
> 表明一个包含值(value)的列表将会传递到构造器中,当然也可以使用引用(ref)

	```xml
	<bean id="beatlesDiscography"
	      class="soundsystem.Discography">
	  <constructor-arg value="The Beatles" />
	  <constructor-arg>
	    <list>
	      <ref bean="sgtPeppers" />
	      <ref bean="whiteAlbum" />
	      <ref bean="hardDaysNight" />
	      <ref bean="revolver" />
	      ...
	    </list>
	  </constructor-arg>
	</bean>
	```
	> 当构造器参数类型是 `java.util.list`时,使用 `<List>`合情合理的.我们也可以使用 `<set>` 元素.最重要的不同在于当Spring创建要装配的集合时，所创建的是`java.util.Set`还是`java.util.List`。如果是Set的话，所有重复的值都会被忽略掉，存放顺序也不会得以保证。不过无论在哪种情况下，`<set>`或`<list>`都可以用来装配`List`、`Set`甚至数组
## 设置属性
前面都是通过构造器注入的,没有使用属性 setter 方法.

```java
public class CDPlayer implements MediaPlayer {

    private CompactDisc cd;
//    public CDPlayer(CompactDisc aa) {
//        this.cd = cd;
//    }
    @Autowired
    public void setCompactDisc(CompactDisc cd){
        this.cd=cd;
    }
    public void play() {
        cd.play();
    }
}

```

该选择构造器注入还是属性注入?
> 强依赖使用构造器注入,对可选性的依赖使用属性注入.

上面的代码,没有构造方法,但是在创建 bean 的时候不会有任何,但是 CDPlayerTest 会出现 NullPointerException 而导致测试失败,因为我们并没有注入 CDPlayer 的 compactDisc 属性.如何注入呢?

```xml
<bean id="cdPlayer"
      class="top.blogcode.CDPlayer">
  <property name="compactDisc" ref="compactDisc" />
</bean>
```
`<property>` 元素为属性的 Setter 方法所提供能与 `<constructor-arg>` 元素为构造器所提供的功能是一样的.

前面我们提供了 c-命名空间,与之类似,Spring 提供了更加简洁的 p-命名空间,作为 <property> 元素的替代方案

```xml

<!-- 第一部分 -->
xmlns:p="http://www.springframework.org/schema/p"

<!-- 第二部分 -->

<bean id="cdplayer" class="top.blogcode.pojo.CDPlayer" p:compactDisc-ref="sgtpeppers"/>

```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g24ii6wbloj308703iwen.jpg)

### 将字面量注入到属性中

```java
package soundsystem;
import java.util.List;
import soundsystem.CompactDisc;

public class BlankDisc implements CompactDisc {

  private String title;
  private String artist;
  private List<String> tracks;

  public void setTitle(String title) {
    this.title = title;
  }

  public void setArtist(String artist) {
    this.artist = artist;
  }


  public void setTracks(List<String> tracks) {
    this.tracks = tracks;
  }

  public void play() {
    System.out.println("Playing " + title + " by " + artist);
    for (String track : tracks) {
      System.out.println("-Track: " + track);
    }
  }

}
```

如果类变成了 setter 的样式,并添加了 list(或set)

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc">
  <property name="title"
               value="Sgt. Pepper's Lonely Hearts Club Band" />
  <property name="artist" value="The Beatles" />
  <property name="tracks">
    <list>
      <value>Sgt. Pepper's Lonely Hearts Club Band</value>
      <value>With a Little Help from My Friends</value>
      <value>Lucy in the Sky with Diamonds</value>
      <value>Getting Better</value>
      <value>Fixing a Hole</value>
      <!-- ...other tracks omitted for brevity... -->
    </list>
  </property>
</bean>
```

如果使用另一种方案 p-命名空间的属性来完成该功能.

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      p:title="Sgt. Pepper's Lonely Hearts Club Band"
      p:artist="The Beatles">
  <property name="tracks">
    <list>
      <value>Sgt. Pepper's Lonely Hearts Club Band</value>
      <value>With a Little Help from My Friends</value>
      <value>Lucy in the Sky with Diamonds</value>
      <value>Getting Better</value>
      <value>Fixing a Hole</value>
      <!-- ...other tracks omitted for brevity... -->
    </list>
  </property>
</bean>
```

> p-命名空间的装配 bean 引用与装配字面量的唯一区别在于是否带有"-ref".如果没有"-ref"后缀,所装配的就是字面量.

> 注意:我们不能使用 p-命名空间来装配`集合`,没有便利的方式使用p-命名空间来指定一个值(或 bean 引用)的列表. 但是,我们可以使用 Spring util-命名空间中的功能来简化.

```xml
<!-- 第一部分 -->

xmlns:util="http://www.springframework.org/schema/util"

<!-- 第二部分 -->
<util:list id="trackList">
  <value>Sgt. Pepper's Lonely Hearts Club Band</value>
  <value>With a Little Help from My Friends</value>
  <value>Lucy in the Sky with Diamonds</value>
  <value>Getting Better</value>
  <value>Fixing a Hole</value>
  <!-- ...other tracks omitted for brevity... -->
</util:list>
```
> `<util:list>` 元素,它会创建一个列表 bean. 借助 `<util:list>`,我们可以将它声明到单独的 bean 之中.将它像使用其他的 bean 那样,将这个列表 bean 注入

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      p:title="Sgt. Pepper's Lonely Hearts Club Band"
      p:artist="The Beatles"
      p:tracks-ref="trackList" />
```

Spring util-命名空间中的元素

|元素|描述|
|:--:|:--:|
|<util:constant>|引用某个类型的 public static 域,并将其暴露为 bean|
|util:list|创建一个 java.util.List 类型的 bean,其中包含值或引用|
|util:map|创建一个 java.util.Map 类型的 bean,其中包含值或引用|
|util:properties|创建一个 java.util.Properties 类型的 bean|
|util:property-path|引用一个 bean 的属性(或内嵌属性),并将其暴露为 bean|
|util:set|创建一个 java.util.Set 类型的 bean,其中包含值或引用|


# 导入和混合配置
在典型的 Spring 应用中,我们可能会同时使用`自动化`和`显式配置`.即便你更喜欢通过 JavaConfig 实现显式配置,但有的时候 XML 却是最佳的方案

幸好在 Spring 中,这些配置方案都不是互斥的.你尽可能将 JavaConfig 的组件扫描和自动装配和/或 XML 配置混合在一起.

**混合配置**
- 第一件需要了解的事情就是在自动装配时,我并不在意装配的 bean 来自哪里(不管是 JavaConfig 或 XML 中)

## 在 JavaConfig 中引用 XML 配置

如果 JavaConfig bean 有些多,我们进行拆分.

1. 从 JavaConfig 拆分出来,定义到自己的类中.
2. 使用 `@Import` 导入到第二个 JavaConfig.


第二种:
创建一个更高级别的 JavaConfig , 在这个类中使用 `@Import` 将两个配置类组合在一起.


假设我们通过 XML 来配置拆分出来的 bean

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      c:_0="Sgt. Pepper's Lonely Hearts Club Band"
      c:_1="The Beatles">
  <constructor-arg>
    <list>
      <value>Sgt. Pepper's Lonely Hearts Club Band</value>
      <value>With a Little Help from My Friends</value>
      <value>Lucy in the Sky with Diamonds</value>
      <value>Getting Better</value>
      <value>Fixing a Hole</value>
      <!-- ...other tracks omitted for brevity... -->
    </list>
  </constructor-arg>
</bean>
```
> 现在是既有 XML 又有 JavaConfig 如何让 Spring 同时加载它和其他基于 Java 的配置呢?

> 使用 `@ImportResource` 注解,假设 拆分的 xml 文件名 "cd-config.xml" 该文件位于根类路径下,那么可以修改上面那个混合两个 JavaConfig 的类.

```java
package soundsystem;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.ImportResource;

@Configuration
@Import(CDPlayerConfig.class)
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig {
}
```

两个 bean,配置在 JavaConfig 中的 CDPlayer 以及配置在 XML 中的 BlankDisc 都会被加载到 Spring 容器. 因为CDPlayer中带有@Bean注解的方法接受一个CompactDisc作为参数，因此BlankDisc将会装配进来，此时与它是通过XML配置的没有任何关系。

## 在 XML 配置中引用 JavaConfig
如果 XML 太多 bean 想要进行拆分.在 JavaConfig 配置中,我们使用 `@Import` 和 `@ImportResource` 来拆分 JavaConfig 类.在 XML 中,我们可以使用 import 元素来拆分 XML 配置

假设希望将 bean 拆分到自己的配置文件中,该文件名为 cd-config.xml,我们可以通过 XML 使用 `<import>` 元素来导入该文件

```xml
<import resource="blank-config.xml"/>
```

如果现在这个 xml 的 bean 转换成 JavaConfig 中,然后将这个 JavaConfig 加入到 xml 中.**事实上,<import> 元素只能导入其他的 XML 配置文件,并没有 XML 元素能够导入 JavaConfig 类**

但是,有一个你以及熟悉的元素能够用来将 Java 配置导入到 XML 配置中: `<bean>`元素.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="top.blogcode.pojo.CDConfig" />

        <bean id="cdPlayer"
              class="top.blogcode.pojo.CDPlayer"
              c:cd-ref="sgetppers" />

</beans>

```

> 还可以创建一个更高层次的配置文件,这个文件不声明任何 bean ,只负责将两个或更多的配置组合起来.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean class="soundsystem.CDConfig" />

  <import resource="cdplayer-config.xml" />

</beans>
```

不管使用JavaConfig还是使用XML进行装配，我通常都会创建一个根配置(rootconfiguration)，也就是这里展现的这样，这个配置会将两个或更多的装配类和/或XML文件组合起来。我也会在根配置中启用组件扫描通过`<context:component-scan>`或`@ComponentScan`
