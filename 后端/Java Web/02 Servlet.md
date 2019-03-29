# Servlet
- 服务端小程序:serve applet
- 特殊的 JAVA 类 (规范不同)
- Servlet 与 Http

以下是 servlet demo 方法也是生命周期;
```java

public class ServletDemo extends HttpServlet {
    @Override
    public void init() throws ServletException {
        System.out.println("=============init 无参==========");
        super.init();
    }

    @Override
    public void init(ServletConfig config) throws ServletException {
        System.out.println("=============init 无参==========");
        super.init(config);
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("=============service ==========");
        PrintWriter out = resp.getWriter();
        out.print("hello world");
        out.close();
    }

    @Override
    public void destroy() {
        System.out.println("=============destroy ==========");
        super.destroy();
    }
}

// tomcat 打印
// =============init 无参==========
// =============init 无参==========
// =============service ==========
```


## Servlet 两种映射方式
- web.xml
    ``` xml
    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>entity.ServletDemo</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
    ```
- 注解
    ```java
    @WebServlet("/hello")
    ```

# Servlet 处理流程分析

> tomcat 既是 `servlet 容器`也是有外部服务器的功能(处理静态页面的功能)

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1jymae56ij30i70c2glr.jpg)

1. 客户端发送请求到 tomcat web server
2. web server 接收请求给 servlet 容器
3. servlet 容器加载servlet,实例后加载req,resp 处理
4. 转发给其他 servlet 处理
5. 将处理结果返回给游览器

## Servlet 生命周期

init(调用一次) -> service(调用多次,看请求次数) -> destory(调用一次)


# Servlet 包介绍
full.document

index.html 查看
了解Servlet常用接口和类

- `javax.servlet`
	- Servlet
		- GenericServlet
		- ServetInputStrem
		- ServletOutputStrem
	- ServletRequest
	- Servlet Content
	- Servlet Config
- `javax.servlet.http`
	- HttpServletRequest
		- getParameter(String key)
		- getParameterValues(String key)
		- getParameterMap()
		- getParameterNames
	- HttpServletRespnse
	- HttpServlet
	- Cookie
- `javax.servlet.annotation`
- `javax.servlet.descriptor`

# 过滤器

数据的过滤 Demo

```java

public class FilterDemo implements Filter {

    public FilterDemo() {
        System.out.println("==========构造函数===========");
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("==========初始化方法===========");
        String initParams=filterConfig.getInitParameter("param");
        System.out.println("=======initParams:"+initParams);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("==========开始过滤===========");
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("==========结束过滤===========");
    }

    @Override
    public void destroy() {
        System.out.println("==========销毁方法===========");
    }
}


```

## 过滤器两种方式

1. web.xml
    ```xml
    <filter>
       <filter-name>filterdemo</filter-name>
       <filter-class>entity.FilterDemo</filter-class>
       <init-param>
           <param-name>param</param-name>
           <param-value>helloWorld</param-value>
       </init-param>
   </filter>

   <filter-mapping>
       <filter-name>filterdemo</filter-name>
       <!-- url-pattern 用来过滤的url -->
       <url-pattern>/*</url-pattern>
   </filter-mapping>
    ```
2. 注解
    ```java
    @WebFilter(value = "/*",initParams = {
            @WebInitParam(name = "param",value = "hellofilter")
    })
    ```
