# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [理解视图解析](#理解视图解析)
- [创建 JSP 视图](#创建-jsp-视图)
	- [配置适用于 JSP 的视图解析器](#配置适用于-jsp-的视图解析器)
		- [解析 JSTL 视图](#解析-jstl-视图)
	- [使用 Spring 的 JSP 库](#使用-spring-的-jsp-库)
		- [将表单绑定到模型上](#将表单绑定到模型上)
		- [展现错误](#展现错误)
		- [Spring 通用的标签库](#spring-通用的标签库)
		- [展现国际化信息](#展现国际化信息)
		- [创建 URL](#创建-url)
		- [转义内容](#转义内容)
- [使用Apache Tiles视图定义布局](#使用apache-tiles视图定义布局)
	- [配置 Tiles 视图解析器](#配置-tiles-视图解析器)
- [使用 Thymeleaf](#使用-thymeleaf)

<!-- /TOC -->
# 理解视图解析
我们所编写的控制器方法都没有直接产生游览器中渲染所需的 HTML。 这些方法只是将一些数据填充到模型中,然后将模型传递给一个用来渲染的视图。这些方法会返回一个 String 类型的值，这个值是视图的逻辑名称，不会直接引用具体的视图实现。

如果控制器中的方法直接负责产生 HTML 的话，就很难在不影响请求处理逻辑的前提下，维护和更新视图。控制器方法和视图的实现会在模型内容上达成一致，这是两者的最大关联，除此之外，两者应该保持足够的距离

但是，如果控制器只通过逻辑视图名来了解视图的话，那 Spring 该如何确定使用哪一个视图实现来渲染模型?前面我们一直使用 InternalResourceResolver 的视图解析器。在它的配置中，为了得到视图的名字，会使用 "/WEB-INF/views/"前缀和".jsp"后缀,从而确定来渲染模型的 JSP 文件的物理位置。

Spring MVC 定义了一个名为 ViewResolver 的接口,它大致如下:

```java
public interface ViewResolver {
  View resolveViewName(String viewName, Locale locale)
                       throws Exception;
}
```

当给 resolveViewName() 方法传入一个视图名和 Locale 对象时,它会返回一个 View 实例。View 是另外一个接口

```java
public interface View {

	String RESPONSE_STATUS_ATTRIBUTE = View.class.getName() + ".responseStatus";

	String PATH_VARIABLES = View.class.getName() + ".pathVariables";

	String SELECTED_CONTENT_TYPE = View.class.getName() + ".selectedContentType";

	@Nullable
	default String getContentType() {
		return null;
	}
	void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
			throws Exception;

}
```
View 接口的任务就是接受模型以及 Servlet 的 request 对象，并将输出结果渲染到 response 中。

Spring 提供了多个内置的实现。

|视图解析器|描述|
|:-:|:-:|
|BeanNameViewResolver|将视图解析为 Spring 应用上下文中的 bean ，其中 bean 的 ID 与视图的名字相同|
|ContentNegotitaingViewResolver|通过考虑客户端需要的内容类型来解析视图，委托给另外一个能够产生对应内容类型的视图解析器|
|FreeMarkerViewResolver|将视图解析为 FreeMarker 模板|
|InternalResourceViewResolver|将视图解析为 Web 应用的内部资源(一般为 JSP)|
|JasperReportsViewResolver|将视图解析为 JasperReports 定义|
|ResourceBundleViewResolver|将视图解析为资源 bundle(一般为属性文件)|
|TilesViewResolver|将视图解析为 Apache Tile 定义，其中 tile ID 与视图名称相同。注意有两个不同的 TilesViewResolver 实现，分别对应于 Tiles 2.0 和 Tiles 3.0|
|UrlBasedViewResolver|直接根据视图的名称解析视图，视图的名称会匹配|
|VelocityLayoutViewResolver |将视图解析为 Velocity 布局，从不同的 Velocity 模板中组合页面|
|VelocityViewResolver|将视图解析为 Velocity 模板|
|XmlViewResolver|将视图解析为特定 XML 文件中的 bean 定义。类似于 BeanName-ViewResolver|
|XsltViewResolver|将视图 XSLT 转换后的结果|

对于这 13 种视图解析器来讲,每一项都对应 Java Web 应用中特定的某种视图技术。InternalResourceViewResolver 一般会用于 JSP ，TilesViewResolver 用于Apache Tiles 视图，而FreeMarkerViewResolver  和 VelocityViewResolver 分别对应 FreeMarker 和 Velocity 模板视图。

后面我们介绍常用到的 JSP，InternalResourceViewResolver,然后介绍 TilesViewResolver,控制 JSP 页面的布局。最后介绍 Thymeleaf 代替 JSP 的技术。

# 创建 JSP 视图
JSP(JavaServer Pages) 作为 Java Web 应用程序的视图技术15年了。Spring 提供了两种支持 JSP 视图的方式
- InternalResourceViewResolver 会将视图名解析为 JSP 文件。另外，如果在你的 JSP 页面中使用了 JSP **标准标签库**(JavaServer Pages Standard Tag Library,JSTL)的话，InternalResourceViewResolver 能够将视图名解析为 JstlView 形式的 JSP 文件,从而将 JSTL 本地化和资源  bundle 变量暴露给 JSTL 的格式化(formatting) 和 信息化(message) 标签。

- Spring 提供了两个 JSP 标签库,一个用于表单到模型的绑定,另一个提供了通用的工具类特性

不管你使用 JSTL ，还是准备使用 Spring 的 JSP 标签库,配置解析 JSP 的视图解析器都是非常重要的。尽管 Spring 还有其他的几个视图解析器都能将视图名映射为 JSP 文件,但就这项任务来讲。

## 配置适用于 JSP 的视图解析器
有些视图解析器,如 ResourceBundleViewResolver 会直接将逻辑视图名映射为特定的 View 接口实现。而 InernalResourceViewResolver 所采取的方法并不那么直接。它遵循一种约定,会在视图名上添加**前缀**和**后缀**，进而确定一个 Web 应用中视图资源的物理路径。


> 场景:一个逻辑视图的路径就是逻辑视图名 home .通用的实践是将 JSP 文件放到 Web 应用 的 WEB-INF 目录下,防止对它的直接访问.如果我们将所有的 JSP 文件都放在 "/WEB-INF/views/"目录下,并且 home 页的 JSP 名为 home.jsp, 那么我们可以确定物理视图的路径就是逻辑视图名 home 再加上 "/WEB-INF/views/" 前缀和 ".jsp" 后缀

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2jclmwrjdj305002wglj.jpg)

当使用 `@Bean` 注解的时候，我们可以按照如下的方式配置 InternalResourceViewResolver ,使其在解析视图时，遵循上述约定

```Java
@Bean
public ViewResolver viewResolver() {
  InternalResourceViewResolver resolver =
      new InternalResourceViewResolver();
  resolver.setPrefix("/WEB-INF/views/");
  resolver.setSuffix(".jsp");
  return resolver;
}
```
还可以使用基于 XML 的 Spring 配置,那么可以按照如下的方式配置：

```xml
<bean id="viewResolver"
      class="org.springframework.web.servlet.view.
                        InternalResourceViewResolver"
      p:prefix="/WEB-INF/views/"
      p:suffix=".jsp" />
```

配置就绪之后,它就会将逻辑视图名解携 JSP 文件,如下:
1. home 将会解析为 "/WEB-INF/views/home.jsp"
2. productList 将会解析为 "/WEB-INF/views/productList.jsp"
3. books/detail 将会解析为 "/WEB-INF/views/books/detail.jsp"

第三个。当逻辑视图名中包含斜线时,这个斜线也会带到资源的路径名中。因此，它会对应到 prefix 属性所引用目录的子目录下的 JSP 文件。这样我们很方便将视图模板组织为层级目录结构，而不是将它们放到同一个目录之中。

### 解析 JSTL 视图

但是如果这些JSP使用JSTL标签来处理格式化和信息的话，那么我们会希望 InternalResourceViewResolver 将视图解析为JstlView。JSTL 的格式化标签需要一个 Locale 对象，以便于恰当地格式化地域相关的值，如日期和货币。信息标签可以借助 Spring 的信息资源和 Locale，从而选择适当的信息渲染到 HTML 之中。通过解析 JstlView，JSTL能够获得 Locale 对象以及 Spring 中配置的信息资源。

> 如果想让 InternalResourceViewResolver 将视图解析为 JstlView， 而不是 InernalResourceView 的话,那么我们只需设置它的 ViewClass 属性即可:

```java
@Bean
public ViewResolver viewResolver() {
  InternalResourceViewResolver resolver =
      new InternalResourceViewResolver();
  resolver.setPrefix("/WEB-INF/views/");
  resolver.setSuffix(".jsp");
  resolver.setViewClass(
      org.springframework.web.servlet.view.JstlView.class);
  return resolver;
}
```

同样也可以使用 XML 完成这一任务；

```xml
<bean id="viewResolver"
      class="org.springframework.web.servlet.view.
                        InternalResourceViewResolver"
      p:prefix="/WEB-INF/views/"
      p:suffix=".jsp"
      p:viewClass="org.springframework.web.servlet.view.JstlView" />
```
不管使用 Java 配置还是使用 XML， 都能确保 JSTL 的格式化和信息标签能够获得 Locale 对象以及 Spring 中配置的信息资源。

## 使用 Spring 的 JSP 库

标签库是一种很强大的方式,能够避免在脚本块中直接编写 Java 代码。 Spring 提供了两个 JSP 标签库。用来帮助定义 Spring MVC Web 的视图。 其中一个标签库会用来渲染 HTML 表单标签,这些标签可以绑定 Model 中的某个属性。另外一个标签库包含了一些工具类标签。


### 将表单绑定到模型上

Spring 的表单绑定 JSP 标签库包含了 14 个标签,它们中的大多数都用来渲染 HTML 中的表单标签。但是，它们与原生 HTML 标签的区别在于它们会绑定模型中的一个对象,能够根据模型中对象的属性填充值。标签库中国还包含了一个为用户展现错误的标签,它会将错误信息渲染到最终的 HTML 之中。

使用 表单 jstl 声明:
```html
<%@ taglib url="http://www.springframework.org/tags/form" profix="sf" %>
```
> 这里前缀指定为"sf"，通常使用 "form" 前缀。这个可以任意。"sf" 主要使用起来简短

|JSP 标签|描述|
|:-:|:-:|
|`<sf:checkbox>`|渲染成一个 HTML `<input>` 标签,其中 type 属性设置为 checkbox|
|`<sf:checkboxs>`|渲染成多个 HTML `<input>`标签,其中 type 属性设置为 checkbox|
|`<sf:errors>`|在一个HTML `<span>` 中渲染输入域的错误|
|`<sf:form>`|渲染成一个 HTML `<form>` 标签,并为其内部标签暴露绑定路径，用于数据绑定|
|`<sf:hidden>`|渲染成一个 HTML `<input>` 标签,其中 type 属性设置为 hidden|
|`<sf:input>`|渲染成一个 HTML `<input>` 标签,其中 type 属性设置为 text|
|`<sf:label>`|渲染成一个 HTML `<label>` 标签|
|`<sf:option>`|渲染成一个 `<option>` 标签,其 selected 属性根据所绑定的值进行设置|
|`<sf:options>`|按照绑定的集合、数组或 Map,渲染成一个 HTML `<option>` 标签的列表|
|`<sf:password>`|渲染成一个 HTML `<input>` 标签,其中 type 属性设置为 password|
|`<sf:radiobutton>`|渲染成一个 HTML `<input>` 标签,其中 type 属性设置为 radio|
|`<sf：radiobuttons>`|渲染成多个 HTML `<input>` 标签,其中 type 属性设置为 radio|
|`<sf:select>`|渲染成一个 HTML `<select>`标签|
|`<sf:textarea>`|渲染成一个 HTML `<textarea>` 标签|

原先 spitter 表达使用 jstl form：

```xml
<sf:form method="POST" commandName="spitter">
  First Name: <sf:input path="firstName" /><br/>
  Last Name: <sf:input path="lastName" /><br/>
  Email: <sf:input path="email" /><br/>
  Username: <sf:input path="username" /><br/>
  Password: <sf:password path="password" /><br/>
  <input type="submit" value="Register" />
</sf:form>
```
通过 commandName 属性构建针对某个 模型对象的上下文信息。在其他的表单绑定标签中，会引用这个模型对象的属性。

> 这里从 spring mvc 5.0 后将 commandName 替换成 modelAttribute ，否则会出现 `Unable to find setter method for attribute: [commandName]`错误

> 在之前的代码中，我们将 modelAttribute 属性设置为 spitter 因此，在模型中必须要有一个key 为 spitter 的对象，否则的话，表单不能正常渲染。这需要修改一下 SpitterController,以确保模型中存在以 spitter 为 key 的 Spitter 对象:

```Java
@RequestMapping(value = "/register",method = RequestMethod.GET)
  public String showRegistration(Model model){
      model.addAttribute(new Spitter());
      return "registerForm";
  }
```

> Spring 3.1 开始,<sf:input> 标签能够允许添加type，date，ragnge和email；对于校验失败后,表单中会预先填充之前输入的值。但是，依然没有告诉用户错在什么地方。为了指导用户矫正错误，我们需要使用 <sf:errors>

### 展现错误
如果存在校验错误的话，请求中会包含错误的详细信息，这些信息是与模型数据放到一起的。我们所需要做的就是到模型中将这些数据抽取出来，并展现给用户。

```xml
<sf:form method="POST" modelAttribute="spitter">
  First Name: <sf:input path="firstName" />
    <sf:errors path="firstName" /><br/>
...
</sf:form>
```

如果 firstName 属性没有错误的话，那么 `<sf:errors>` 不会渲染任何内容。但如果有校验错误的话，那么它将会在一个 HTML <span> 标签中显示错误信息。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2jiyglky8j309n07v74h.jpg)

为了修改错误样式，可以设置 cssClass 属性:

```html
<sf:form method="POST" commandName="spitter" >
  First Name: <sf:input path="firstName" />
    <sf:errors path="firstName" cssClass="error" /><br/>
...
</sf:form>
```

```css
span.error{
    color:red;
}
```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2jj2ucztej309y07lt8t.jpg)

另外一种处理校验错误方式就是将所有的错误信息在同一个地方进行显示。为了做到这一点，我们可以移除每个输入域上的`<sf:errors>`元素，并将其放到表单的顶部，
```html
<sf:form method="POST" commandName="spitter" >
  <sf:errors path="*" element="div" cssClass="errors" />

...
</sf:form>
```

path `"*"` 是通配符,会告诉 `<sf:errors>` 展现所有属性的错误。同样需要注意的是，我们将element属性设置成了div。默认情况下，错误都会渲染在一个HTML `<span>`标签中，如果只显示一个错误的话，这是不错的选择。但是，如果要渲染所有输入域的错误的话，很可能要展现不止一个错误，这时候使用`<span>`标签（行内元素）就不合适了。像`<div>`这样的块级元素会更为合适。因此，我们可以将element属性设置为div，这样的话，错误就会渲染在一个`<div>`标签中。

```css
div.errors{
    color:red;
}
```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2jj8sdxwjj309j0acglx.jpg)


现在，我们在表单的上方显示所有的错误，这样页面布局可能会更加容易一些。但是，我们还没有着重显示需要修正的输入域。通过为每个输入域设置cssErrorClass属性，这个问题很容易解决。我们也可以将每个 label 都替换为`<sf:label>`，并设置它的 cssErrorClass 属性。如下就是做完必要修改后的 FirstName 输入域：

```html
<sf:form method="POST" commandName="spitter" >
  <sf:label path="firstName"
      cssErrorClass="error">First Name</sf:label>:
    <sf:input path="firstName" cssErrorClass="error" /><br/>
...
</sf:form>
```

如果它所绑定的属性有任何错误的话，在渲染得到的`<label>`元素中，class属性将会被设置为error，如下所示：

```html
<label for="firstName" class="error">First Name</label>
```
```css
label.error{
    color: red;
}
input.error{
    background-color: #ffcccc;
}
```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2jjgztk1yj309p07qdfy.jpg)

> 如果对错误信息不满意，可以在校验注解上设置 message 属性,使其引用对用户更友好的信息,而这些信息可以定义在属性文件中:


```Java
@NotNull
@Size(min=5, max=16, message="{username.size}")
private String username;

@NotNull
@Size(min=5, max=25, message="{password.size}")
private String password;

@NotNull
@Size(min=2, max=30, message="{firstName.size}")
private String firstName;

@NotNull
@Size(min=2, max=30, message="{lastName.size}")
private String lastName;

@NotNull
@Email(message="{email.valid}")
private String email;
```

创建一个ValidationMessages.properties 的文件(这里文件名必须保持一致，放入 resources):
```txt
firstName.size=
    First name must be between {min} and {max} characters long.
lastName.size=
    Last name must be between {min} and {max} characters long.
username.size=
    Username must be between {min} and {max} characters long.
password.size=
    Password must be between {min} and {max} characters long.
email.valid=The email address must be valid.
```
每条信息的 key 值对应于注解中 message 属性占位符的值。

### Spring 通用的标签库

```html
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
```

|JSP 标签|描述|
|:-:|:-:|
|`<s:bind>`|将绑定属性的状态导出到一个名为 status 的页面作用域属性中,与`<s:path>`组合使用获取绑定属性的值|
|`<s:escapeBody>`|将标签体中的内容进行 HTML 和/或 JavaScript 转义|
|`<s:hasBindErrors>`|根据指定模型对象(在请求属性中)是否有绑定错误,有条件地渲染内容|
|`<s:hasEscape>`|为当前页面设置默认的 HTML 转义值|
|`<s:message>`|根据给定的编码获取信息，然后要么进行渲染，要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域的变量(通过使用 var 和 scope 属性实现)|
|`<s:nestedPath>`|设置嵌入式的 path，用于 `<s:bind>` 之中|
|`<s:theme>`|根据给定的编码主题获取信息，然后要么进行渲染，要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域的变量(通过使用 var 和 scope 属性实现)|
|`<s:transform>`|使用命令对象的属性编辑器转换命令对象中不包括的属性|
|`<s:url>`|创建相对于上下文的 URL，支持 URI 模板变量以及 HTML/XML/JavaScript 转义.可以渲染 URL (默认行为) ,要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域的变量(通过使用 var 和 scope 属性实现)|
|`<s:eval>`|计算符合 Spring 表达式语言(Spring Expression Language,SpEL) 语法的某个表达式的值,然后要么进行渲染(默认行为),要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域的变量(通过使用 var 和 scope 属性实现)|

一些标签已经被Spring表单绑定标签库淘汰了。例如，<s:bind>标签就是Spring最初所提供的表单绑定标签，它比我们在前面所介绍的标签复杂得多。因为这些标签库的行为比表单绑定标签少得多，所以我不会详细介绍每个标签

> flag:敲了半天，说不用。。。。。

### 展现国际化信息

对于渲染文本来说，是很好的方案，文本能够位于一个或多个属性文件中。借助`<s:message>`，我们可以将硬编码的欢迎信息替换为如下的形式:

```html
<h1>Welcome to Spittr!</h1>

<h1><s:message code="spittr.welcome" /></h1>
```
按照这里的方式 `<s:message>` 将会根据 key 为 spitter。welcome 的信息来渲染文本。因此,我们为 `<s:message>` 配置一个信息源

> Spring 有多个信息源的类，它们都实现了 MessageSource 接口。在这些类中，更为常见和又用的是 ResourceBundleMessageSource.它会从一个属性文件中加载信息。
```java
@Bean
public MessageSource messageSource() {
  ResourceBundleMessageSource messageSource =
          new ResourceBundleMessageSource();
  messageSource.setBasename("messages");
  return messageSource;
}
```

在这个bean声明中，核心在于设置 basename 属性。你可以将其设置为任意你喜欢的值，在这里，我将其设置为 message 。将其设置为 message 后，ResourceBundleMessageSource 就会试图在根路径的属性文件中解析信息，这些属性文件的名称是根据这个基础名称衍生得到的。


ReloadableResourceBundleMessageSource ,但是它能够重新加载信息属性,而不必重新编译或重启应用。如下是配置 ReloadableResourceBundleMessageSource 的样例:

```java
@Bean
public MessageSource messageSource() {
  ReloadableResourceBundleMessageSource messageSource =
      new ReloadableResourceBundleMessageSource();
  messageSource.setBasename("file:///etc/spittr/messages");
  messageSource.setCacheSeconds(10);
  return messageSource;
}
```

这里的关键区别在于 basename 属性设置为在应用的外部查找（而不是像 ResourceBundleMessageSource 那样在类路径下查找）。 basename 属性可以设置为在类路径下（以"classpath:"作为前缀）、文件系统中（以 "file:" 作为前缀）或 Web应用的根路径下（没有前缀）查找属性。在这里，我将其配置为在服务器文件系统的"/etc/spittr"目录下的属性文件中查找信息，并且基础的文件名为 "messages"

> 最后创建 messages.properties 文件
```txt
spitter.welocome=Welcome to Spitter

/* 创建中文欢迎信息 messages_zh.properties */
spitter.welcome = 欢迎哦！
```
> flag：尝试做一个 demo

### 创建 URL

`<s:url>` 是一个很小的标签。它主要的任务就是创建 URL,然后将其赋值给一个变量或渲染到响应中。它是 JSTL 中 `<c:url>` 标签的替代者,但是它具备几项特殊的技巧。

按照其最简单的形式，`<s:url>`会接受一个相对于Servlet上下文的URL，并在渲染的时候，预先添加上Servlet上下文路径。例如，考虑如下`<s:url>`的基本用法：

```html
<a href="<s:url href="/spitter/register" />">Register</a>
<!-- 会被渲染成: -->
<a href="/spittr/spitter/register">Register</a>

```
这样,我们在创建 URL 的时候,就不必再担心 Servlet 上下文路径是什么,`<s:url>` 将会负责这件事。


`<s:url>`创建 URL 并将其赋值给一个变量供模板在稍后使用:
```html
<s:url href="/spitter/register" var="registerUrl" />
<a href="${registerUrl}">Register</a>
```

默认情况下，URL 是在页面作用域内创建的。但是通过设置 scope 属性，我们可以让`<s:url>`在应用作用域内、会话作用域内或请求作用域内创建 URL:

```HTML
<s:url href="/spitter/register" var="registerUrl" scope="request" />
```

如果希望在 URL 上添加参数的话,那么你可以使用 `<s:param>` 标签。比如,如下 `<s:url>`使用两个内嵌的 `<s:param>` 标签,来设置 "/spittles" 的 max 和 count 参数:

```html
<s:url href="/spittles" var="spittlesUrl">
  <s:param name="max" value="60" />
  <s:param name="count" value="20" />
</s:url>

<!-- url 结果 -->
register?max=10&min=20
```
如果我们需要创建带有路径(path) 参数的 URL

```html
<s:url href="/spitter/{username}" var="spitterUrl">
  <s:param name="username" value="jbauer" />
</s:url>
```
当 href 属性中的占位符匹配`<s:param>`中所指定的参数时，这个参数将会插入到占位符的位置中。如果`<s:param>`参数无法匹配 href 中的任何占位符，那么这个参数将会作为**查询参数**。

`<s:url>` 标签还可以解决 URL 得到转义需求。例如,如果你希望将渲染得到 URL 内容展现在 Web 页面上(而不是作为超链接),那么你应该要求 `<s:url>` 进行 HTML 转义,这需要将 htmlEscape 属性设置为 true。例如,如下的 `<s:url>` 将会渲染 HTML 转义后的 URL:

```html
<s:url value="/spittles" htmlEscape="true">
  <s:param name="max" value="60" />
  <s:param name="count" value="20" />
</s:url>
```
希望在 JavaScirpt 代码中使用 URL 的话,那么应该将 javaScript-Escape 属性设置为 true:

```html
<s:url value="/spittles" var="spittlesJSUrl" javaScriptEscape="true">
  <s:param name="max" value="60" />
  <s:param name="count" value="20" />
</s:url>
<script>
  var spittlesUrl = "${spittlesJSUrl}"
</script>

<!-- js中显示为 -->

 var spittlesUrl = "\/spitter\/spittles?max=60&count=20"

```

### 转义内容
`<s:escapeBody>`标签是一个通用的转义标签。它会渲染标签体中内嵌的内容,并且在必要的时候进行转义。例如，假设你希望在页面上展现一个 HTML 代码片段。为了正确显示,我们需要将"<"和">" 字符替换为 "`&lt;`"和"`&gt;`",当然可以手动，但是这很繁琐,并且代码难以阅读。

```html
<s:escapeBody htmlEscape="true">
<h1>Hello</h1>
</s:escapeBody>
```
它将会在响应体中渲染成如下的内容:

```
&lt;h1&gt;Hello&lt;/h1&gt;
```

# 使用Apache Tiles视图定义布局
> 场景: 如果我们有很多页面，但是想拥有相同的模板样式,如果页面很多扩展性和维护就比较难。

更好的方式式使用布局引擎,如 Apache Tiles,定义适用于所有页面的通用页面布局。Spring MVC 以视图解析器的形式为 Apache Tiles 提供了支持,这个视图解析器能够将逻辑视图名解析为 Tile 定义

## 配置 Tiles 视图解析器
为了在 Spring 中使用 Tiles,需要配置几个 bean。

- 需要一个 TilesConfigurer bean,它会负责定位和加载 Tile 定义并协调生成 Tiles。
- 需要 TilesViewResolver bean 将逻辑视图名称解析为 Tile 定义。

两个组件两种形式:
- Apache Tiles 2 TilesConigurer/TilesViewResolver位于 **org.springframework.web.servlet.view.titles2**
- Apache Tiles 3 TilesConigurer/TilesViewResolver位于 **org.springframework.servlet.view.tiles3**


1. **导包**

```xml
<!-- https://mvnrepository.com/artifact/org.apache.tiles/tiles-jsp -->
<dependency>
    <groupId>org.apache.tiles</groupId>
    <artifactId>tiles-jsp</artifactId>
    <version>3.0.7</version>
</dependency>
<dependency>
    <groupId>org.apache.tiles</groupId>
    <artifactId>tiles-core</artifactId>
    <version>3.0.7</version>
</dependency>
<dependency>
    <groupId>org.apache.tiles</groupId>
    <artifactId>tiles-api</artifactId>
    <version>3.0.7</version>
</dependency>

```
2. **定义tiles.xml 文件**

```xml
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">
<tiles-definitions>

  <definition name="base"
              template="/WEB-INF/layout/page.jsp">
    <put-attribute name="header"
                   value="/WEB-INF/layout/header.jsp" />
    <put-attribute name="footer"
                   value="/WEB-INF/layout/footer.jsp" />
  </definition>

  <definition name="home" extends="base">
    <put-attribute name="body"
                   value="/WEB-INF/views/home.jsp" />
  </definition>

  <definition name="registerForm" extends="base">
    <put-attribute name="body"
                   value="/WEB-INF/views/registerForm.jsp" />
  </definition>

  <definition name="profile" extends="base">
    <put-attribute name="body"
                   value="/WEB-INF/views/profile.jsp" />
  </definition>

  <definition name="spittles" extends="base">
    <put-attribute name="body"
                   value="/WEB-INF/views/spittles.jsp" />
  </definition>

  <definition name="spittle" extends="base">
    <put-attribute name="body"
                   value="/WEB-INF/views/spittle.jsp" />
  </definition>

</tiles-definitions>

```

3. **模板page.jsp**

```xml
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="t" %>
<%@ page session="false" %>
<html>
  <head>
    <title>Spittr</title>
    <link rel="stylesheet"
          type="text/css"
          href="<s:url value="/resources/style.css" />" >
  </head>
  <body>
    <div id="header">
      <t:insertAttribute name="header" />
    </div>
    <div id="content">
      <t:insertAttribute name="body" />
    </div>
    <div id="footer">
      <t:insertAttribute name="footer" />
    </div>
  </body>
</html>

```

4. **配置bean**
    - java
    ```java
    @Bean
    public TilesConfigurer tilesConfigurer() {
      TilesConfigurer tiles = new TilesConfigurer();
      tiles.setDefinitions(new String[] {
          "/WEB-INF/layout/tiles.xml"
      });
      tiles.setCheckRefresh(true);
      return tiles;
    }

    ```
    - xml
    ```xml
    <bean id="tilesConfigurer" class=
        "org.springframework.web.servlet.view.tiles3.TilesConfigurer">
      <property name="definitions">
        <list>
          <value>/WEB-INF/layout/tiles.xml.xml</value>
          <value>/WEB-INF/views/**/tiles.xml</value>
        </list>
      </property>
    </bean>
    <bean id="viewResolver" class=
        "org.springframework.web.servlet.view.tiles3.TilesViewResolver" />
    ```
5. **配置视图解析器**

```java
@Bean
    public ViewResolver viewResolver(){
        return new TilesViewResolver();
    }
```

# 使用 Thymeleaf
