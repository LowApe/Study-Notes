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

## 练习1. 简单登录
> 很简单,所以讲一下,就是一个jsp 跳转到一个 servlet ;<br>需要注意的是,uri 的路径=`"request.getContextPath()"`+"/xxx" 或,不加`/`默认之前的添加,"/"表示跟目录

## 练习2. 处理 GET 和  POST 请求
|操作|GET|POST|
|:----:|:----:|
|刷新|不会重复提交|重复提交|
|数据长度|2048 字符|无限制|
|数据类型|ASCII 字符|无限制|
|可见性|URL中可见|URL中不可见|
|安全性|差|高|

## 练习3. 转发和重定向

转发:
    1. 只能请求转发给相同应用地址,不能是网站地址

```java
    RequestDispathcer rd= request.getRequestDispatcher("uri");
    rd.forword(request,response);
```

重定向:
    1. 先进行带参的 servlet 访问将参数加到uri末尾?xx=xx&xx=xx;
    2. 然后进行指定页面访问

```java
    response.sendRedirect("www.blogcode.top");
```



> 从代码可以看出,请求转发是`Request`,可以添加属性进行传递,但是重定向是`response`

## 练习4: jdbc+dao+controller+转发




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
# 过滤器链
- 当有两个过滤器,通过 web.xml 注册顺序决定过滤器的执行顺序,这是一种[责任链设计模式]()
    ```
    ==========2开始过滤===========
    ==========开始过滤===========
    ==========结束过滤===========
    ==========2结束过滤===========
    ```
- 当两个过滤器,通过注解,则包内的顺序,就是执行顺序
    ```
    ==========开始过滤===========
    ==========2开始过滤===========
    ==========2结束过滤===========
    ==========结束过滤===========
    ```


## 练习1:通过过滤器进行编码设置

1. init 初始化获取config的字符编码配置
2. 定义一个字符集变量
3. doFilter 获取字符的变量是否等于请求的,
4. 执行过滤链接  

```java
String initParams=null;

@Override
   public void init(FilterConfig filterConfig) throws ServletException {
       System.out.println("==========初始化方法===========");
       initParams=filterConfig.getInitParameter("charset");
       if(initParams ==null){
           throw new ServletException("EncodingFilter 的编码设置为空!");
       }
       System.out.println("=======initParams:"+initParams);
   }

@Override
   public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
       System.out.println("==========开始过滤===========");
       HttpServletRequest httpServletRequest = (HttpServletRequest)servletRequest;
       HttpServletResponse httpServletResponse = (HttpServletResponse)servletResponse;
       /*
       * 当前设置与请求字符编码不一致,当前
       * */
       if(!initParams.equals(httpServletRequest.getCharacterEncoding())){
           httpServletRequest.setCharacterEncoding(initParams);

       }
       httpServletResponse.setCharacterEncoding(initParams);
       System.out.println("initParams="+initParams);
       filterChain.doFilter(servletRequest,servletResponse);
       System.out.println("==========结束过滤===========");
   }
```

### 练习2:登录校验
1. 完成基本登录调用
2. 登录成功记录 session ,并记录登录 uri ,如果有 uri 跳转,没有就跳转首页
3. 首页判断 session 显示退出和登录
4. 退出清除 seesion 跳转首页


* 注意: uri 路径
