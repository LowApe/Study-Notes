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
