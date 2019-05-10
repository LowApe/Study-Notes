# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [Spring MVC配置的替代方案](#spring-mvc配置的替代方案)
	- [自定义 DispacherServlet 配置](#自定义-dispacherservlet-配置)
	- [添加其他的 Servlet 和 Filter](#添加其他的-servlet-和-filter)
	- [在 web.xml 中声明 DispacherServlet](#在-webxml-中声明-dispacherservlet)
- [处理 multipart 形式的数据](#处理-multipart-形式的数据)
	- [配置 multipart 解析器](#配置-multipart-解析器)
		- [使用 Servlet 3.0 解析 multipart 请求](#使用-servlet-30-解析-multipart-请求)
		- [配置 Jakarta Commons FileUpload multipart 解析器](#配置-jakarta-commons-fileupload-multipart-解析器)
	- [处理 multipart 请求](#处理-multipart-请求)
		- [接受 MultipartFile](#接受-multipartfile)
		- [将文件保存到 Amazon S3 中](#将文件保存到-amazon-s3-中)
		- [以 Part 的形式接受上传的文件](#以-part-的形式接受上传的文件)
- [处理异常](#处理异常)
	- [将异常映射为 HTTP 状态码](#将异常映射为-http-状态码)
	- [编写异常处理的方法](#编写异常处理的方法)
- [为控制器添加通知](#为控制器添加通知)
- [跨重定向请求传递数据](#跨重定向请求传递数据)
	- [通过 URL 模板进行重定向](#通过-url-模板进行重定向)
	- [使用 flash 属性](#使用-flash-属性)

<!-- /TOC -->
# Spring MVC配置的替代方案

之前通过扩展 AbstractAnnotationConfigDispatcherServletInitialer 快速搭建了 Spring MVC 环境。除了 DispatcherServlet ,我们可能还需要额外的 Servlet 和 Filter ；可能需要将应用部署到 Servlet 3.0 之前的容器中,那需要传统的 web.xml

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
借助 customizeRegistration() 方法中的 ServletRegistration.Dynamic ,我们能够完成多项任务,包括通过调用 setLoadOnStartup() 设置 load-on-startup 优先级,通过 setInitParameter() 设置初始化参数,通过调用 setMultipartConfig() 配置 Servlet3.0 对 multipart 的支持。在前面的样例中,我们设置了对 multipart 的支持,将上传文件的临时存储目录设置在"/tmp/spittr/uploads"中

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

其实 AbstractAnnotationConfigDispatcherServletInitialer 提供了一个方法,实现添加 Filter

```java
@Override
protected Filter[] getServletFilters() {
  return new Filter[] { new MyFilter() };
}
```

如果要将应用部署到 Servlet3.0 容器中,那么 Spring 提供了多种方式来注册 Servlet （包括DispatcherServlet）、Filter和Listener,而不必创建 web.xml 文件。但是,如果你不想采取以上所述方案的话,也是可以的。假设你需要将应用部署到**不支持** Servlet3.0 的容器中（或者你只是希望使用web.xml文件）,那么我们完全可以按照传统的方式,通过web.xml配置 SpringMVC 。让我们看一下该怎么做

## 在 web.xml 中声明 DispacherServlet

在典型的 SpringMVC 应用中,我们会需要 DispatcherServlet 和 ContextLoaderListener。AbstractAnnotationConfigDispatcherServletInitializer 会自动注册它们,但是如果需要在 web.xml 中注册的话,那就需要我们自己来完成这项任务了。如下是一个基本的 web.xml 文件,它按照传统的方式搭建了 DispatcherServlet 和 ContextLoaderListener

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

DispatcherServlet 会根据Servlet的名字找到一个文件,并基于该文件加载应用上下文。如果你希望指定 DispacherServlet 配置文件的位置的话,那么可以在 Servlet 上指定一个 contextConfigLocation 初始化参数。例如,如下的配置中,DispatcherServlet 会从 "/WEB-INF/spring/appServlet/servlet-context.xml" 加载它的bean；

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
> flag：我的理解是,将不同应用的 bean 存放在 xml 中

要在 SpringMVC 中使用基于 Java 的配置,我们需要告诉 DispatcherServlet和 ContextLoaderListener 使用 AnnotationConfigWebApplicationContext ,这是一个 WebApplicationContext 的实现类,它会加载Java配置类,而不是使用XML。要实现这种配置,我们可以设置contextClass上下文参数以及DispatcherServlet的初始化参数。


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
在 Web 应用中,允许用户上传内容是很常见的需求。Spittt 应用需要两个地方文件上传。一个注册上传图片.一个表单上传。

表单提交所形成的请求以 "&" 符分割的多个 `name-value`对:

```txt
firstName=Charles&lastName=Xavier&email=professorx%40xmen.org
&username=professorx&password=letmein01
```
尽管这种编码形式很简单,但对于传送二进制数据,比如上传图片,就显得力不从心。 **multipart** 格式的数据会将一个表单拆分为多个部分,每个部分对应的一个输入域。在意般的表单输入域中,它所对应的部分中会放置文本型数据,但如果上传文件的话,它所对应的部分可以是二进制,下面展现了 multipart 的请求体:

```shell
------WebKitFormBoundaryqgkaBn8IHJCuNmiW
Content-Disposition: form-data; name="firstName"

Charles
------WebKitFormBoundaryqgkaBn8IHJCuNmiW
Content-Disposition: form-data; name="lastName"

Xavier
------WebKitFormBoundaryqgkaBn8IHJCuNmiW
Content-Disposition: form-data; name="email"

charles@xmen.com
------WebKitFormBoundaryqgkaBn8IHJCuNmiW
Content-Disposition: form-data; name="username"

professorx
------WebKitFormBoundaryqgkaBn8IHJCuNmiW
Content-Disposition: form-data; name="password"

letmein01
------WebKitFormBoundaryqgkaBn8IHJCuNmiW
Content-Disposition: form-data; name="profilePicture"; filename="me.jpg"
Content-Type: image/jpeg
// (二进制数据)
  [[ Binary image data goes here ]]
------WebKitFormBoundaryqgkaBn8IHJCuNmiW--

```
在这个 multipart 的请求中,我们可以看到 profilePicture 部分与其他部分明显不同。除了其他内容以外,它还有自己的 Content-Type头,这表明它是一个 JPEG 图片。不一定那么明显,但是 profilePicture 部分的请求体是二进制数据.

在 Spring MVC 中编写控制器方法处理文件上传之前,我们必须配置一个 multipart 解析器,通过它来告诉 DispatcherServlet 该如何读取 multipart 请求。

## 配置 multipart 解析器
DispacherServlet 并没有实现任何解析 multipart 请求数据的功能.它将任务委托给了 Spring 中 MultipartResolver 策略接口实现。通过这个接口解析 multipart 请求中的内容。内置两个 MultipartResolver:
- CommonsMultipartResolver：使用 Jakarta Commons FileUpload 解析 multipart 请求;
- StandardServletMultipartResolver：依赖于 Servlet 3.0 对 multipart 请求的支持,优先使用这个,它使用 Servlet 所提供的功能支持,并不需要依赖任何其他的项目。如果部署到 Servlet 3.0 之前可能使用 CommonsMultipartResolver

### 使用 Servlet 3.0 解析 multipart 请求
StandardServletMultipartResolver 没有构造器参数,也没有设置的属性。
```java
@Bean
public MultipartResolver multipartResolver() throws IOException {
  return new StandardServletMultipartResolver();
}
```

通过 multipart 配置来限定存储路径,文件大小等,临时路径是必须配置的,可以通过 web.xml 或 Servlet 初始化类中,将 multipart 具体细节作为 DispatcherServlet 配置的一部分

```java
DispatcherServlet ds = new DispatcherServlet();
Dynamic registration = context.addServlet("appServlet", ds);
registration.addMapping("/");
registration.setMultipartConfig(
    new MultipartConfigElement("/tmp/spittr/uploads"));
```

如果配置 DispacherServlet 的 Servlet 初始化类继承了 AbstractAnnotationConfigDispatcherServletInitialer或 AbstractDispacherServletInitializer 的话,将不会对 Dynamic Servlet registartion 的引用供我们使用了。但是可以通过重载 customizeRegistration() 方法(它会得到一个 Dynamic 作为参数)来配置 multipar 的具体细节:

```java
@Override
  protected void customizeRegistration(ServletRegistration.Dynamic registration) {
      registration.setMultipartConfig(new
       MultipartConfigElement("/tmp/spitter/uploads"));
  }
```
MultipartConfigElement 一个参数的构造器指定文件体统中的一个绝对路径,上传文件将会临时写入该目录中。但是,我们还可以通过其他的构造器来限制上传文件的大小。除了临时路径的位置,其他的构造器所能接受的参数如下
- 上传文件的最大容量(以字节为单位),默认没有限制
- 整个 multipart 请求的最大容量(以字节为单位),不会关心有多少个 part 以及每个 part 的大小。默认是没有限制的
- 在上传的过程中,如果文件大小达到了一个指定最大容量(以字节为单位),将会写入到临时文件路径中。默认值为 0。

加入限制文件大小不超过 2MB,整个请求不超过 4MB,而且所以的文件都能写到磁盘中.

```java
@Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setMultipartConfig(new
                MultipartConfigElement("/tmp/spitter/uploads"
        ,2097152,4194304,0));
    }
```

使用 web.xml 来配置的话 MultipartConfigElement 的话,那么可以使用 `<servlet>` 中的 `<multipart-config>` 元素

```xml
<servlet>
  <servlet-name>appServlet</servlet-name>
  <servlet-class>
    org.springframework.web.servlet.DispatcherServlet
  </servlet-class>
  <load-on-startup>1</load-on-startup>
  <multipart-config>
    <location>/tmp/spittr/uploads</location>
    <max-file-size>2097152</max-file-size>
    <max-request-size>4194304</max-request-size>
  </multipart-config>
</servlet>
```
`<multipart-config>`的默认值与 MultipartConfigElement 相同。与 MultipartConfigElement一样,必须要配置的事 `<location>`

> flag：不知道为什么没有这个标签元素。

### 配置 Jakarta Commons FileUpload multipart 解析器

通常来讲,StandardServletMultipartResolver 会是最佳的选择,如果部署到非 Servlet 3.0 的容器中,那么就得需要替代的方案。如果喜欢的话,我们可以编写自己的 MultipartResolver 实现。如果不是处理 multipart 请求执行特定的逻辑,否则没有必要。

Spring 内置了 CommonsMultipartResolver,可以作为 StandardServletMultipartResolver 替代方案。

```java
@Bean
public MultipartResolver multipartResolver() {
  return new CommonsMultipartResolver();
}
```
与 StandardServletMultipartResolver 有所不同,CommonsMultipartResolver不会强制要求设置临时文件路径。默认情况下,这个路径就是 Servlet 容器的临时目录。不过,通过设置uploadTempDir属性,我们可以将其指定为一个不同的位置：

```java
@Bean
public MultipartResolver multipartResolver() throws IOException {
  CommonsMultipartResolver multipartResolver =
      new CommonsMultipartResolver();
  multipartResolver.setUploadTempDir(
      new FileSystemResource("/tmp/spittr/uploads"));
  return multipartResolver;
}
```

通过是设置 CommonsMultipartResolver 的属性。指定其他 multipart 上传细节:

```java
@Bean
public MultipartResolver multipartResolver() throws IOException {
  CommonsMultipartResolver multipartResolver =
          new CommonsMultipartResolver();
  multipartResolver.setUploadTempDir(
      new FileSystemResource("/tmp/spittr/uploads"));
  multipartResolver.setMaxUploadSize(2097152);
  multipartResolver.setMaxInMemorySize(0);
  return multipartResolver;
}
```

在这里,我们将最大的文件容量设置为2MB,最大的内存大小设置为 0 字节。这两个属性直接对应于 MultipartConfigElement 的第二个和第四个构造器参数,表明不能上传超过 2MB 的文件,并且不管文件的大小如何,所有的文件都会写到磁盘中。但是与 MultipartConfigElement 有所不同,我们无法设定 multipart 请求整体的最大容量。

## 处理 multipart 请求
已经配置好了对 mutipart 请求的处理,那么接下来我们就可以编写控制器方法来接受上传的文件,最常见的方式就是在某控制器方法参数上添加 `@RequestPart` 注解显允许用户在注册的时候上传一张照片,修改之前的表单：

```html
<form method="POST" th:object="${spitter}"
      enctype="multipart/form-data">

...
  <label>Profile Picture</label>:
    <input type="file"
           name="profilePicture"
           accept="image/jpeg,image/png,image/gif" /><br/>

...

</form>
```

`<form>` 标签现在将 enctype 属性设置为 `multipart/form-data`,这回告诉游览器以 multipart 数据的形式提交表单,而不是以表单数据的形式进行提交。在 multipart 中,每个输入域都会对应一个 part。上面 html 很好理解

现在,我们需要修改 processRegistration() 方法,使其能够接受上传的图片.其中一种方式是添加 byte 数组参数,并为其添加 `@RequestPart` 注解。

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @RequestPart("profilePicture") byte[] profilePicture,
    @Valid Spitter spitter,
    Errors errors) {
  ...
}
```
如果没有提交文件,byte 数组为空。获取到图片数据后, processRegistration() 方法剩下的任务就是将文件保存到某个位置

### 接受 MultipartFile

Spring 提供 MultipartFile 接口,它为处理 multipart 数据提供了内容更为丰富的对象。

```java
package org.springframework.web.multipart;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;

public interface MultipartFile {
  String getName();
  String getOriginalFilename();
  String getContentType();
  boolean isEmpty();
  long getSize();
  byte[] getBytes() throws IOException;
  InputStream getInputStream() throws IOException;
  void transferTo(File dest) throws IOException;
}
```

MultipartFile 提供了获取上传文件 byte 的方式,但是它所提供的功能并不仅限于此,还能获得原始的文件名,大小以及内容类型。 **它还提供一个 InputStream,用来将文件数据以流的方式进行读取**

transferTo() 方法,它能够帮助我们将上传的文件写入到文件系统中。

```java
 profilePicture.transferTo(
     new File("/data/spittr/" + profilePicture.getOriginalFilename()));
```

将文件保存到本地文件系统中是非常简单的,但是这需要我们对这些文件进行管理。我们需要确保有足够的空间,确保当出现硬件故障时,文件进行了备份,还需要在集群的多个服务器之间处理这些图片文件的同步。

### 将文件保存到 Amazon S3 中

另一种方案就是让别人来负责处理这些事情。多几行代码,我们就能将图片保存到云端。例如:

```java
private void saveImage(MultipartFile image)
    throws ImageUploadException {
  try {
    AWSCredentials awsCredentials =
        new AWSCredentials(s3AccessKey, s2SecretKey);
    // 构建 S3 服务
    S3Service s3 = new RestS3Service(awsCredentials);
    // 创建 S3 bucker 和 object
    S3Bucket bucket = s3.getBucket("spittrImages");
    S3Object imageObject =
        new S3Object(image.getOriginalFilename());

    //设置图片数据
    imageObject.setDataInputStream(
        image.getInputStream());
    imageObject.setContentLength(image.getSize());
    imageObject.setContentType(image.getContentType());

    //设置权限
    AccessControlList acl = new AccessControlList();
    acl.setOwner(bucket.getOwner());
    acl.grantPermission(GroupGrantee.ALL_USERS,
        Permission.PERMISSION_READ);
    imageObject.setAcl(acl);
    // 保存图片
    s3.putObject(bucket, imageObject);
  } catch (Exception e) {
    throw new ImageUploadException("Unable to save image", e);
  }
}

```

这个 saveImage() 方法所做的第一件事就是构建 Amazon Web Service (AWS)凭证。为了完成这一点,你需要有一个 S3 Access Key 和 S3 Secret Access Key。当注册 S3 服务的时候,Amazon 会将其提供给你。它们会通过值注入的方式提供给 Spitter-Controller。

AWS凭证准备好后,saveImage()方法创建了一个JetS3t的RestS3Service实例,可以通过它来操作S3文件系统。它获取spitterImagesbucket的引用并创建用来包含图片的S3Object对象,接下来将图片数据填充到S3Object。


在调用putObject()方法将图片数据写到S3之前,saveImage()方法设置了S3Object的权限,从而允许所有的用户查看它。这是很重要的——如果没有它的话,这些图片对我们应用程序的用户就是不可见的。最后,如果出现任何问题的话,将会抛出ImageUploadException异常。

### 以 Part 的形式接受上传的文件

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    @RequestPart("profilePicture") Part profilePicture,
    @Valid Spitter spitter,
    Errors errors) {
  ...
}
```

上面通过 javax.servlet.http.Part 作为控制器方法的参数。下面展现 Part 方法
于 MultipartFile 并没有太大的差别。
```java
package javax.servlet.http;
import java.io.*;
import java.util.*;

public interface Part {
  public InputStream getInputStream() throws IOException;
  public String getContentType();
  public String getName();
  public String getSubmittedFileName();
  public long getSize();
  public void write(String fileName) throws IOException;
  public void delete() throws IOException;
  public String getHeader(String name);
  public Collection<String> getHeaders(String name);
  public Collection<String> getHeaderNames();
}
```

差别比如:getSubmittedFileName() 对应于 getOriginalFilename().类似第 write() 对应于 transferTo（）

```java
profilePicture.write("/data/spittr/" +
        profilePicture.getOriginalFilename());
```

值得一提的是，如果在编写控制器方法的时候，通过 Part 参数的形式接受文件上传，那么就没有必要配置 MultipartResolver了。只有使用MultipartFile的时候，我们才需要 MultipartResolver。

# 处理异常
Spring 提供了多种方式将异常转换为响应:
- 特定的 Spring 异常将会自动映射为指定的 HTTP 状态码
- 异常上可以添加 `@ResponseStatus` 注解,从而将其映射为某一个 Http 状态码
- 在方法上可以添加 `@ExceptionHandler` 注解,使其用来处理异常。

## 将异常映射为 HTTP 状态码
在默认情况下,Spring 会将自身的一些异常自动转换为适合的状态码。

|Spring 异常| HTTP 状态码|
|:--:|:--:|
|BindExcpetion|400 - Bad Request|
|ConversionNotSupportedException|500 - Internal Server Error|
|HttpMediaTypeNotAcceptableException|406 - Not Acceptable|
|...|...|

通过 `@ResponseStatus` 注解将异常映射为 HTTP 状态码。

```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
    @PathVariable("spittleId") long spittleId,
    Model model) {
  Spittle spittle = spittleRepository.findOne(spittleId);
  if (spittle == null) {
    throw new SpittleNotFoundException();
  }
  model.addAttribute(spittle);
  return "spittle";
}
```

如果出现任意没有映射的异常,响应都会带有 500 状态码,但是我们通过自定义异常精确响应状态码 404

这里添加一个异常处理类
```java
package spittr.web;
@ResponsStatus(value=HttpStatus.NOT_FOUND,
reason="Spittle NOT_FOUND")
public class SpittleNotFoundException extends RuntimeException {
}
```
在引入 `@ResponseStatus` 注释之后,如果控制器方法抛出 SpittleNotFound-Exception 异常的话,响应将会具有 404 状态,这是因为而 Spittle Not Found

## 编写异常处理的方法
如果我们不仅包括状态码,还要包括所处理的错误,这就不能将 HTTP 视为错误,而是要按照处理请求的方式处理异常。

```java
@ExceptionHandler(DuplicateSpittleException.class)
public String handleDuplicateSpittle() {
  return "error/duplicate";
}
```

handleDuplicateSpittle() 方法上添加了`@ExceptionHandller`,当发生此类的异常,返回一个 String 类型,这与处理请求的方法一致的,指定了要渲染的逻辑视图名,它能够告诉他们正在视图创建一条重复的条目

# 为控制器添加通知
如果在多个控制器类中都会抛出某个特定的异常,那么你可能会发现要在所有的控制器方法中重复相同的 `@ExceptionHandlder` 方法。为了避免重复,我们会创建一个基础的控制器类,所有控制器类或者这个类,从而继承通用 `@ExceptionHandler` 方法。

Spring 3.2 为这类问题引入了一个新的解决方案:**控制器通知**。控制器通知(controller advice) 是任意带有 `@ControllerAdvice` 注解的类,这个类会包含一个或多个如下类型的方法:
- `@ExceptionHandler 注解标注的方法:`
- `@InitBinder 注解标注的方法:`
- `@ModelAttribute 注解标注的方法:`

在带有 `@ControllerAdvice` 注解的类中,以上所述的这些方法会运用到整个应用程序所有控制器中带有 `@RequestMapping` 注解的方法上

```java
package spitter.web;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class AppWideExceptionHandler {

  @ExceptionHandler(DuplicateSpittleException.class)
  public String duplicateSpittleHandler() {
    return "error/duplicate";
  }

}

```

> flag:这边书上并没有运行效果啥的,总感觉模模糊糊的


# 跨重定向请求传递数据
前面执行 POST 请求后,通常来讲一个最佳实践就是执行一下重定向.除了其他的一些因素外,这样做能够防止用户点击游览器的刷新按钮或后退箭头时,客户端重新执行危险的 POST 请求。

> 问题:正在发起重定向功能的方法该如何发送数据给重定向的目标方法呢?

答:一般来讲，当一个处理器方法完成之后，该方法所指定的模型数据将会复制到请求中，并作为请求中的属性，请求会转发(forward)到视图上进行渲染。因为控制器方法和视图所处理的是同一个请求，所以在转发的过程中，请求属性能够得以保存。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2w1ise6i7j30b502l0su.jpg)

对于重定向来说,模板并不能用来传递数据。
- 使用 URL 模板以路径变量和 / 或查询参数的形式的形式传递数据
- 通过 flash 属性发送属性

## 通过 URL 模板进行重定向
```java
return "redirect:/spitter/{username}"
```

> 注意:当构建 URL 或 SQL 查询语句的适合,使用 String 连接是很危险的。

改写请求转发：

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, Model model) {
  spitterRepository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  return "redirect:/spitter/{username}";
}
```
如果有两个查询参数

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, Model model) {
  spitterRepository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  model.addAttribute("spitterId", spitter.getId());
  return "redirect:/spitter/{username}";
}
```
最后重定向路径将会是 `/spitter/bahuma?spitterId=41`
但是使路径变量和查询参数只能处理简单数据。

## 使用 flash 属性

如果我们发送不是参数,而是要发送实际的 Spitter 对象。但是对象要比参数复杂.模型数据最终以请求参数的形式复制到请求中的，当重定向发送的时候,数据就会的丢失.因此,我们需要将 Spitter 对象放到一个位置,使其能够在重定向的过程中存活下来。

- 将Spitter 放到会话中,得到后清理会话
- Spring 提供了将数据发送为 flash 属性(flash attribute) 的功能.flash 属性会一直携带这些数据直到下一次请求,然后才会消失

```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(
    Spitter spitter, RedirectAttributes model) {
  spitterRepository.save(spitter);
  model.addAttribute("username", spitter.getUsername());
  model.addFlashAttribute("spitter", spitter);
  return "redirect:/spitter/{username}";
}
```

通过对模型对象添加呆 addAttribute() 方法添加 flash,原理是在重定向执行之前,所有的flash属性都会复制到会话.重定向后,存在会话中的 flash 属性会被取出,并从会话转移到模型之中

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2w2d4l9flj30aw04fwex.jpg)
