# 目录

# Spring MVC配置的替代方案

之前通过扩展 AbstractAnnotationConfigDispatcherServletInitialer 快速搭建了 Spring MVC 环境。除了 DispatcherServlet ，我们可能还需要额外的 Servlet 和 Filter ；可能需要将应用部署到 Servlet 3.0 之前的容器中,那需要传统的 web.xml

## 自定义 DispacherServlet 配置

AbstractAnnotationConfigDispatcherServletInitialer 必须要重载三个方法,但实际在 AbstractAnnotationConfigDispatcherServletInitialer 将 DispacherServlet 注册到 Servlet 容器中之后,就会调用 customizeRegistration(),并将 Servlet 注册后得到的 Registration.Dynamic 传递进来。**通过重载 customizeRegistration()方法,我们可以对 DispacherServlet 进行额外的配置**

> 可以查看AbstractAnnotationConfigDispatcherServletInitialer上两级的源码

```java
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(
      new MultipartConfigElement("/tmp/spittr/uploads"));
}
```
借助 customizeRegistration() 方法中的 ServletRegistration.Dynamic ，我们能够完成多项任务，包括通过调用 setLoadOnStartup() 设置 load-on-startup 优先级，通过 setInitParameter() 设置初始化参数，通过调用 setMultipartConfig() 配置 Servlet3.0 对 multipart 的支持。在前面的样例中，我们设置了对 multipart 的支持，将上传文件的临时存储目录设置在“/tmp/spittr/uploads”中

## 添加其他的 Servlet 和 Filter
按照 AbstractAnnotationConfigDispatcherServletInitialer 的定义它会创建 DispatcherServlet 和 ContextLoaderListener.但是如果想注册其他的 Servlet,Filter 或 Listener如何实现?

基于 Java 的初始化器(initializer)的一个好处就在于我们可以定义任意数量的初始化器类。因此,如果我们想往 Web 容器中注册其他组件的话,只需创建一个新的初始化器就可以了。最简单的方式就是实现 Spring 的 webApplicationInitializer 接口

```java
package com.myapp.config;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration.Dynamic;
import org.springframework.web.WebApplicationInitializer;
import com.myapp.MyServlet;

public class MyServletInitializer implements WebApplicationInitializer {

  @Override
  public void onStartup(ServletContext servletContext)
         throws ServletException {
             //注册 Servlet
    Dynamic myServlet =
        servletContext.addServlet("myServlet", MyServlet.class);
            //映射 Servlet
    myServlet.addMapping("/custom/**");
  }

}

```

上面代码注册了一个 Servlet 并将其映射到一个路径上.我们也可以通过这种方式来手动注册 DispatcherServlet。类似我们可以创建新的 WebApplicationInitializer 实现来注册 Listener 和 Filter

```java
@Override
public void onStartup(ServletContext servletContext)
       throws ServletException {

  javax.servlet.FilterRegistration.Dynamic filter =
      servletContext.addFilter("myFilter", MyFilter.class);

  filter.addMappingForUrlPatterns(null, false, "/custom/*");
}

```

其实 AbstractAnnotationConfigDispatcherServletInitialer 提供了一个方法，实现添加 Filter

```java
@Override
protected Filter[] getServletFilters() {
  return new Filter[] { new MyFilter() };
}
```

如果要将应用部署到 Servlet3.0 容器中，那么 Spring 提供了多种方式来注册 Servlet （包括DispatcherServlet）、Filter和Listener，而不必创建 web.xml 文件。但是，如果你不想采取以上所述方案的话，也是可以的。假设你需要将应用部署到**不支持** Servlet3.0 的容器中（或者你只是希望使用web.xml文件），那么我们完全可以按照传统的方式，通过web.xml配置 SpringMVC 。让我们看一下该怎么做

## 在 web.xml 中声明 DispacherServlet

在典型的 SpringMVC 应用中，我们会需要 DispatcherServlet 和 ContextLoaderListener。AbstractAnnotationConfigDispatcherServletInitializer 会自动注册它们，但是如果需要在 web.xml 中注册的话，那就需要我们自己来完成这项任务了。如下是一个基本的 web.xml 文件，它按照传统的方式搭建了 DispatcherServlet 和 ContextLoaderListener

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5"
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
<!-- 设置根上下文配置文件位置 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
  </context-param>
<!-- 注册ContextListeneer -->
  <listener>
    <listener-class>
      org.springframework.web.context.ContextLoaderListener
    </listener-class>
  </listener>
<!-- 注册 DispatcherServlet -->
  <servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>
<!-- 将 DisapacherServlet 映射到"/" -->
  <servlet-mapping>
    <servlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>

```
ContextLoaderListener 和 DispatcherServlet 各自都会加载一个 Spring 应用上下文。上下文参数 contextConfigLocation 指定了一个XML 文件的地址,这个文件定义了根应用上下文,它会被 ContextConfigLocationListener 加载。根上下文会从 "/WEB-INF/spring/root-context.xml"中加载 bean 定义

DispatcherServlet 会根据Servlet的名字找到一个文件,并基于该文件加载应用上下文。如果你希望指定 DispacherServlet 配置文件的位置的话,那么可以在 Servlet 上指定一个 contextConfigLocation 初始化参数。例如,如下的配置中,DispatcherServlet 会从 "/WEB-INF/spring/appServlet/servlet-context.xml" 加载它的bena；

```xml
<servlet>
  <servlet-name>appServlet</servlet-name>
  <servlet-class>
    org.springframework.web.servlet.DispatcherServlet
  </servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
      /WEB-INF/spring/appServlet/servlet-context.xml
    </param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
```
> flag：我的理解是，将不同应用的 bean 存放在 xml 中

要在 SpringMVC 中使用基于 Java 的配置，我们需要告诉 DispatcherServlet和 ContextLoaderListener 使用 AnnotationConfigWebApplicationContext ，这是一个 WebApplicationContext 的实现类，它会加载Java配置类，而不是使用XML。要实现这种配置，我们可以设置contextClass上下文参数以及DispatcherServlet的初始化参数。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5"
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

  <context-param>
    <param-name>contextClass</param-name>
    <param-value>
      org.springframework.web.context.support.AnnotatioConfigWebpplicatioContext
    </param-value>
  </context-param>

  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>com.habuma.spitter.config.RootConfig</param-value>
  </context-param>

  <listener>
    <listener-class>
      org.springframework.web.context.ContextLoaderListener
    </listener-class>
  </listener>

  <servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
      <param-name>contextClass</param-name>
      <param-value>
        org.springframework.web.context.support.AnnotationConfigWebpplicatioContext
      </param-value>
    </init-param>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>
        com.habuma.spitter.config.WebConfig
      </param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>

```

> flag：这个 web.xml 可以替换 java 加载配置,对应之前的 `SpittrWebAppInitializer`类那种加载方式

# 处理 multipart 形式的数据
