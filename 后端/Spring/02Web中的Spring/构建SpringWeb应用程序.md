# 目录

Spring MVC 注解来构建处理各种 Web 请求、参数和表单输入的控制器.

# Spring MVC 起步

请求是如何从客户端到 Spring MVC 中的组件,最终返回到客户端的
## Spring MVC 请求路径

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2e9vhfeysj30bk06fmxw.jpg)

1. 请求离开客户端游览器(会带有用户所请求的信息,至少包含 URL)
2. DispatcherServlet 这个前端控制器将请求发送给处理器映射器(handler mapping)查询请求的下一站(根据 URL 选择合适的控制器)
3. 到了控制器,请求会将处理请求附带的信息,等待信息处理完成(设计良好的控制器本身只处理很少或不处理,而是将业务逻辑委托给一个或多个服务对象进行处理-service)
4. 控制器完成逻辑处理后,产生模型(model)信息和逻辑视图名
5. DispatcherServlet 将使用视图解析器(view resolver)来将逻辑视图匹配为一个特定的视图实现
6. DispatcherServlet 交付模型数据给视图,视图将模型数据渲染输出到响应对象传递给客户端

## 搭建 Spring MVC

### 配置 DispatcherServlet
- web.xml 中配置(目前有更简单的,so 不介绍)
- Java 类配置

```java
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class SpittrWebAppInitializer
        extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { WebConfig.class };
    }
}
```
> 如何理解？我们可能只需要知道扩展`AbstractAnnotaionCondfigDispatcherServletInitilaizer` 的任意类都会自动配置 DispatcherServlet 和 Spring 应用上下文,Spring 应用上下文,Spring 的应用上下文会位于应用程序的 Servlet 上下文之中

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2earws4jqj30ew0d1dgc.jpg)

> flag: 暂时看书中理解部分.

实现代码重写了三个方法
1. getServletMappings(),它会将一个或多个路径映射到 DispatcherServlet 上(‘/’表示它会是应用的默认 Servlet)
    > **理解 DispatcherServlet 和 Servlet 监听器(ContextLoaderListener)的关系**

    > 当 DispatcherServlet 启动的时候,它会创建 Spring 应用上下文,并加载配置文件或配置类中所声明的 bean.`getServletConfigClasses()`方法中,我们要求 DispatcherServlet 加载应用上下文时,使用定义在 WebConfig 配置类(使用 Java 配置)中的bean

    > 但是在 Spring Web应用中,通常还会有另外一个应用上下文.另外的这个应用上下文是由 ContextLoaderListener 创建的.

    > 我们希望 DispatcherServlet 加载包含 Web 组件的 bean,如控制器、视图解析器以及处理器映射,而 ContextLoaderListener 要加载应用中的其他 bean.这些 bean 通常是驱动应用后端的中间层和数据层组件.

2. getServletConfigClasses() 方法返回的带有`@Configuration`注解的类将会用来定义 DispatcherServlet 应用上下文中的bean.
3. getRootConfigClasses() 方法返回的带有`@Configuration`注解的类将会用来配置 ContextLoaderListener 创建的应用上下文中的bean.

> 如果按照这种方式配置 DispatcherServlet ,而不是 web.xml ,唯一问题是它只能部署到支持 Servlet 3.0 的服务器中才能正常工作,如 Tomcat 7.0 以上版本

### 启动 Spring MVC

- xml 配置(`<mvc:annotaion-driven>`)
- Java 配置

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc
public class WebConfig {
}
```
`@EnableWebMvc` 注解类启动 Spring MVC,但是：
- 没有配置视图解析器:默认使用 BeanNameViewResolver,这个视图解析器会查找 Id 与视图名称匹配的 bean,并且查找的 bean 要实现 View 接口,它以这样的方式来解析视图.
- 没有启动组件扫描.只能找到在显式声明在配置类中的控制器
- 这样配置会映射默认 Servlet,它会处理所有请求,包括静态资源的请求.

因此,我们需要给 WebConfig 这个最小 Spring MVC 配置上添加内容

```java
package spittr.config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@ComponentScan("package")
public class WebConfig
       extends WebMvcConfigurerAdapter {

  @Bean
  public ViewResolver viewResolver() {
    InternalResourceViewResolver resolver =
            new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    resolver.setExposeContextBeansAsAttributes(true);
    return resolver;
  }

  @Override
  public void configureDefaultServletHandling(
        DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }

}
```
`@ComponentScan` 扫描当前包下的组件.接下来添加一个 ViewResolver bean.它会查找视图资源文件,在查找的时候,它会在视图上一个特定的前缀和后缀(名为 home 的视图会被解析为/WEB-INF/views/home.jsp)

最后重写了 `configureDefaultServletHandling()`方法.DefaultServletHandlerConfigurer的 enable()方法,要求 DispatcherServlet 将对静态资源的请求转发到 Servlet 容器中默认的 Servlet 上,而不是使用 DispatcherServlet 本身来处理此类请求.

WeConfig 就绪那么 RootConfig?

```java
package spittr.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@ComponentScan(basePackages={"spitter"},
    excludeFilters={
        @Filter(type=FilterType.ANNOTATION, value=EnableWebMvc.class)
    })
public class RootConfig {
}
```

# 编写基本的控制器
控制器只是方法上添加注解 `@RequestMapping` 注解的类,这个注解声明了它们所要处理的请求

开始的时候,我们尽可能简单,假设控制器类要处理对"/"的请求,并渲染应用的首页。

```java
import static org.springframework.web.bind.annotation.RequestMethod.*;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class HomeController {
  @RequestMapping(value="/", method = RequestMethod.GET)
  public String home() {
    return "home";
  }
}

```

HomeController 是一个构造型(stereotype)的注解,它基于`@Component`注解。它的目的就是辅助实现组件扫描。因为 HomeController 带有 `@Controller` 注解,因此组件扫描器会自动找到 HomeController ,并将其声明为 Spring 应用上下文中的一个 bean

当然也可以给 HomeController 添加 `@Component` 注解，但是表意性差一些。

唯一的 home（）方法,带有 `@RequestMapping` 注解。它的 value 属性指定了这个方法所要处理的请求路径,method 属性细化它所处理的 Http 方法。方法返回类型是 String 类型 "home" 。这个 home 就是将被 Spring MVC 解读为要渲染的视图名称。

> DispatcherServlet 会要求`视图解析器`将这个`逻辑名`称解析为实际的视图,鉴于我们配置 InternalResourceViewResolver 的方式,视图名 "home" 将会解析为 "WEB-INF/views/home.jsp" 路径

## 测试控制器

```java

import static org.junit.Assert.assertEquals;
import org.junit.Test;
import spittr.web.HomeController;

public class HomeControllerTest {
  @Test
  public void testHomePage() throws Exception {
    HomeController controller = new HomeController();
    assertEquals("home", controller.home());
  }
}
```

> 它完全没有站在 Spring MVC 控制器的视角进行测试.这个测试没有断言当接受到针对"/" 的 GET 请求时会调用 home() 方法。因为而它返回的值就是 "home".Spring 现在包含了一种` mock` Spring MVC  并针对控制器指向 Http 请求的机制。这样的话,在测试控制器时候，没有必要启动 Web 服务器和 Web游览器了

> 我测试了一下控制器的返回值，是对应网页名称，是对应返回值为 String 的方法，方法名貌似只是区别与，不同请求处理调用的方法不同；如果返回值是基本数据类型，而且在控制器中的处理器的模型中不添加 key ，默认是返回类型的小写形式。处理器如果是引用数据类型的呢？
```java

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import staticorg.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import staticorg.springframework.test.web.servlet.setup.MockMvcBuilders.*;
import org.junit.Test;
import org.springframework.test.web.servlet.MockMvc;
import spittr.web.HomeController;

public class HomeControllerTest {
  @Test
  public void testHomePage() throws Exception {
    HomeController controller = new HomeController();
    MockMvc mockMvc =
        standaloneSetup(controller).build();

    mockMvc.perform(get("/"))
           .andExpect(view().name("home"));
  }
}

```

## 定义类级别的请求处理

```java
@Controller
@RequestMapping("/")
public class HomeController {
  @RequestMapping(method = RequestMethod.GET)
  public String home() {
    return "home";
  }
}
```

路径现在被转移到类级别的`@RequestMapping` 上，而 http 方法仍然映射到方法级别，只不过是这个注解会应用控制器中所有的处理方法上

> 注意:整个项目路径映射不能存在冲突

## 传递模型数据到视图中

之前 home 是最简单的控制器，没有任何的数据，现在需要传递模型数据到视图中。这就需要连接到数据库

```java
public interface SpittleRepository {
    /**
     * ID 属性最大值，count 表示返回 Spittle 对象的数量
     * @param max
     * @param count
     * @return
     */
    List<Spittle>  findSpittles(long max,int count);
}

// ==============

@Controller
@RequestMapping( "/spittles")
public class SpittleController {
    private SpittleRepository spittleRepository;
    @Autowired
    public SpittleController(SpittleRepository spittleRepository) {
        this.spittleRepository=spittleRepository;
    }
    @RequestMapping(method = RequestMethod.GET)
    public String spittles(Model model){
        model.addAttribute(spittleRepository.findSpittles(Long.MAX_VALUE,20));
        return "spittles";
    }

}


```

> controller 有一个构造器,这个构造器使用了 `@Autowired` 注解,用来注入 SpittleRepository。这个随后又用在 spittles 方法中,用来获取最新的 spittle 列表。

其中 Model 作为参数，Model 实际上就是 Map （也就是 key-value）它会传递给视图,这样数据就能渲染到客户端了

> 当调用 addAtrribute() 方法并且不指定 key 的时候,那么 key 会根据值的对象类型推断确定。在本例中，因为它是一个 List<Spittle> 因此,建将会推断为 spittleList(响应请求的参数)

jsp 结构

```html
<c:forEach items="${spittleList}" var="spittle" >
  <li id="spittle_<c:out value="spittle.id"/>">
    <div class="spittleMessage">
      <c:out value="${spittle.message}" />
    </div>
    <div>
      <span class="spittleTime"><c:out value="${spittle.time}" /></span>
      <span class="spittleLocation">
          (<c:out value="${spittle.latitude}" />,
          <c:out value="${spittle.longitude}" />)</span>
    </div>
  </li>

</c:forEach>
```

# 接受请求的输入
有些 Web 应用是只读的。人们只能通过游览器在站点查阅数据，但不是一成不变。众多的 Web 应用允许用户参与进去,将数据发送到服务器，也就是客户端发送带参数请求。

Spring MVC 允许多种方式将客户端中的数据传送到控制器的处理器方法中,包括:
- 查询参数(Query Parameter)
- 表单参数(Form Parameter)
- 路径变量(Path Variable)

## 处理查询参数

> 场景:如果我们查看一堆数据，客户端用户向查看历史的数据，并且按照这个顺序显式；有可能有很多页，用户想查看具体的某一页,等等这些，可能需要想服务端发送请求，那如何说明我想要得请求，就可以将这些"需求"参数发送给服务端识别。

> 如果实现分页的功能,我们编写的处理器方法要接受如下的参数:
- before 参数(表明结果中所有 id 在这个值之前)
- count 参数(表明结果中要包含具体返回数量)


```java
@RequestMapping(method = RequestMethod.GET)
public List<Spittle> spittles(
        @RequestParam("max") long max,
        @RequestParam("count") int count
        ){
    return spittleRepository.findSpittles(max,count);
}
```
![](http://ww1.sinaimg.cn/large/006rAlqhly1g2h9b47l8nj30jr0av0sz.jpg)


如何处理带参和不带参的处理呢？
```java
@RequestMapping(method = RequestMethod.GET)
    public List<Spittle> spittles(
            @RequestParam(value = "max",defaultValue ="50000") long max,
            @RequestParam(value = "count",defaultValue = "5") int count
            ){
        return spittleRepository.findSpittles(max,count);
    }
}
```
![](http://ww1.sinaimg.cn/large/006rAlqhly1g2h9d0uq6pj30eb0ia0th.jpg)

> 请求中的查询参数是往控制器中传递信息的常用手段。另外一种流行的方式在构建`面向资源`的控制器时，这种方式就是将`传递参数作为请求路径`的一部分。下面会讲到不同过查询参数，而是使用路径参数。

## 通过路径参数接受输入
如果应用程序需要根据给定的 ID 来展现一个 Spittle 记录。其中一种方案就是编写处理器方法,通过使用 `@RequestParam`注解,让它接受 ID 作为查询参数，下面展示`通过查询参数的方式`

```java
@RequestMapping(value = "/show",method = RequestMethod.GET)
  public  String showSpittle(@RequestParam(value = "spittle_id") long spittleId,Model model){
      model.addAttribute(spittleRepository.findOne(spittleId));
      return "spittle";
  }
```
> 上面就是通过查询参数，这个处理器方法可以处理这样的 `/spittles/show?spittle_id=12345` 请求。`在理想情况下,要识别的资源应该通过 URL 路径进行标示，而不是通过查询参数` 对`/spittles/12345` 发起 GET 请求要优于对 `/spittles/show?spittle_id=12345` 发起请求。

Spring MVC 允许我们在 `@RequestMapping` 路径中添加占位符。占位符的名称要用大括号({})。路径中的其他部分要与所处理的请求 url 完全匹配,但是占位符部分可以是任意值

```java
@RequestMapping(value = "/{spittleId}",method = RequestMethod.GET)
   public  String spittle(@PathVariable("spittleId") long spittleId, Model model){
       model.addAttribute(spittleRepository.findOne(spittleId));
       return "spittle";
   }
```
> 上面代码实例中，如果路径变量的值与参数变量相同，则可以写成这样 `@PathVariable long spittleId`

如果传递请求中少量的数据,那`查询参数`和`路径变量`是很合适的。但通常我们还需要传递很多的数据(表单提交的数据),那查询数据显得有些笨重和受限了。

# 处理表单
Web 应用的功能通常并不局限于向用户推送内容。大多数的应用允许用户填充表单并将数据提交回应用中，通过这种方式实现与用户的交互。像提供内容一样,Spring MVC 的控制器也为表单处理提供了良好的支持。


## 编写处理表单的控制器
当处理注册表单的 POST 请求时,控制器需要接受表单数据据并将表单数据保持为 Spitter 对象。最后，为了防止`重复提交`，应该将游览器重定向到新创建用户的基本信息页面。


> 在处理 POST 类型的表达时，在请求处理完成后，最好进行一下重定向，这样游览器的刷新就不会重复提交表单了。

```java
@RequestMapping(value="/register", method=POST)
 public String processRegistration(Spitter spitter) {
   spitterRepository.save(spitter);

   return "redirect:/spitter/" +
          spitter.getUsername();
 }
```
上面代码会出现null 的路径，是因为我们 post 接受到的是一个 Spitter 对象，但是 post 不能进行对象传送，接受应该使用 HttpRequestServlet;

> 当 InternalResourceViewResolver 看到视图格式中的"redirect:"前缀时，它就知道要将其解析为重定向的规则，而不是视图名称。

> 还能识别"forward:" 请求将会前往(forward) 指定的 URL 路径,而不再是重定向。

上面代码还有不完善的地方，当发生重定向的时候，这个重定向的页面请求还没有加进来

```java
@RequestMapping(value="/{username}", method=GET)
public String showSpitterProfile(
      @PathVariable String username, Model model) {
  Spitter spitter = spitterRepository.findByUsername(username);
  model.addAttribute(spitter);
  return "profile";
}
```

## 校验表单

如果用户提交表单的时候，username 或 password 文本域为空的话，那么将会导致在新建 Spitter 对象中为空。如何处理?

1. 前端限制信息的正确性
2. 后端处理器方法中添加代码检查合法性。

> Spring 对 Java 校验 API (Java Validation API,又称为 JSR-303) 的支持。 从 Spring 3.0 开始，Spring MVC 提供校验 API 支持。所有注解都位于 `javax.validation.constraints` 包

```java
JSR提供的校验注解：         
@Null   被注释的元素必须为 null    
@NotNull    被注释的元素必须不为 null    
@AssertTrue     被注释的元素必须为 true    
@AssertFalse    被注释的元素必须为 false    
@Min(value)     被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@Max(value)     被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@DecimalMin(value)  被注释的元素必须是一个数字，其值必须大于等于指定的最小值    
@DecimalMax(value)  被注释的元素必须是一个数字，其值必须小于等于指定的最大值    
@Size(max=, min=)   被注释的元素的大小必须在指定的范围内    
@Digits (integer, fraction)     被注释的元素必须是一个数字，其值必须在可接受的范围内    
@Past   被注释的元素必须是一个过去的日期    
@Future     被注释的元素必须是一个将来的日期    
@Pattern(regex=,flag=)  被注释的元素必须符合指定的正则表达式    
---------------------
```

校验注解添加到相关的实体类上

```java
private Long id;
   @NotNull
   @Size(min=5,max=16)
   private String firstName;
   @NotNull
   @Size(min=5,max=16)
   private String lastName;
   @NotNull
   @Size(min=2,max=30)
   private String userName;
   @Size(min=2,max=30)
   private String passWord;
```

倚赖信息
```xml
<!-- https://mvnrepository.com/artifact/javax.validation/validation-api -->
       <dependency>
           <groupId>javax.validation</groupId>
           <artifactId>validation-api</artifactId>
           <version>2.0.1.Final</version>
       </dependency>
```

添加玩校验注释,接下来需要修改控制器中处理器方法的来应用校验功能

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @Valid Spitter spitter,
     Errors errors) {

  if (errors.hasErrors()) {
    return "registerForm";
   }

  spitterRepository.save(spitter);
  return "redirect:/spitter/" + spitter.getUsername();
}

```

在参数中添加了 `@Valid`注解，这回告诉 Spring，需要确保这个对象满足校验限制。
> 需要注意 Errors参数要紧跟在 `@Valid` 注释参数之后

> 注意通过bean 来获取表单元素参数，实体类名称需要与表单的name 属性一致

> 还有需要保证在类路径下包含这个 Java API 的实现，`比如 Hibernate Validator`,这边我 Maven 少导入这个包，而且在WEb lib 中少添加了这个包


![](http://ww1.sinaimg.cn/large/006rAlqhly1g2iejajwf6j318d0m1q6w.jpg)
