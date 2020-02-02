# 目录


- Spring Sercurity 介绍
- 使用 Servlet 规范中的 Filter 保护 Web 应用
- 基于数据库和 LDAP 进行认证

# Spring Security简介

Spring Security 是为基于 Spring 的应用程序提供声明式安全保护的安全性框架。 Spring Security 提供了完整的安全性解决方案,它能够在 Web 请求级别**处理身份认证和授权**.Spring Security 充分利用了 DI 和 AOP

## 理解Spring Security的模块
不管使用 Spring Security 保护那种类型的应用程序,第一件事将 Spring Security 模块添加到应用程序的**类路径下**. Spring Security 3.2 分为 11 个模块

|模块|描述|
|:--:|:--:|
|ACL|支持通过访问控制列表(Access control list,ACL) 为域对象提供安全性|
|切面(Aspects)|一个很小的模块,当使用 Spring Security 注解时,会使用基于 AspectJ 的切面,而不是使用标准的 Spring AOP|
|CAS 客户端(CAS Client)|提供与 Jasig 的中心认证服务(Central Authtication Service,CAS)进行集成的功能|
|**配置(Configuration)**|包含通过 XML 和 Java 配置 Spring Security 的功能支持|
|**核心(Core)**|提供 Spring Security 基本库|
|加密(Cryptography)|提供了加密和密码编码的功能|
|LDAP|支持基于 LDAP 进行认证|
|OpenID|支持使用 OpenID 进行集中式认证|
|Remoting|提供了对 Spring Remoting 的支持|
|**标签库(Tag Library)**|Spring Security 的 JSP 标签库|
|**Web**|提供 Spring Security 基于 Filter 的 Web 安全性支持|

应用程序的类路径下至少要包含 Core 和 Configuration 这两个模块。Spring Security 经常被用于保护 Web 应用,这显然也是 Spittr 应用的场景,所有我们还需要添加 Web 模块。同时我们还会用到 Spring Security JSP 标签库,所以这个模块也需要添加使用。

> 使用 maven 需要导入 `spring-security-core`,`spring-security-config`,`spring-security-web`
## 过滤 Web 请求

Spring Security 借助一系列 Servlet Filter 来提供各种安全性功能。你可能会想,这是否意味我们需要在 web.xml 或 WebApplicationInitializer 中配置多个 Filter 呢? **实际上,借助 Spring 的小技巧,只需要配置一个 Filter**。

DelegatingFilterProxy 是一个特殊的 Servlet Filter,它本身所做的工作不多,只是将工作委托给一个 `javax.servlet.Filter`实现类,这个实现类作为一个 `<bean>`注册在 Spring 应用的上下文中:

![](http://ww1.sinaimg.cn/large/006rAlqhly1g3g2bymtxwj30dd025t8p.jpg)

传统 web.xml:

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
        org.springframeworkd.web.filter.DelegatingFilterProxy
    </filter-class>
</filter>
```
这里有一个名为 springSecurityFilterChain 的 Filter bena,DelegatingFilterProxy 会将过滤逻辑委托给它

如果你希望借助 WebApplicationInitializer 以 Java 的方式配置 DegationgFilterChain ,需要创建一个扩展的新类:

```java
public class SecurityWebInitializer
    extends AbstractSecurityWebApplicationInitializer{}
```

AbstractSecurityWebApplicationInitializer 实现了 WebApplicationInitializer,因此 Spring 会发现它,并用它在 Web 容器中注册 DelegatingFilterProxy。 尽管我们可以重载它的 appendFilters()或insertFilters() 方法来注册自己选择的 Filter(查看源码有着两个类),但要注册 ** DelegatingFilterProxy,我们并不需要重载任何方法。**

> 不管我们通过 web.xml 还是通过 AbstractSecurityWebApplicationInitializer 的子类来配置 DelegatingFilterProxy,它都会拦截发往应用中的请求,并将请求委托给 ID 为 springSecurityFilterChain bean。


springSecurityFilterChain 本身是另一个特殊的 Filter,它也被称为 FilterChainProxy。**它可以链接任意一个或多个其他的 Filter，Spring Security 依赖一系列 Servlet Filter 来提供不同的安全特性。但是,几乎不要知道细节.当我们启动 Web 安全性的时候,会自动创建这些 Filter**

## 编写简单的安全性配置

最简单的 java 配置:
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity   // 启动Web 安全性
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}

```

WebSecurityConfigurerAdapter 是 WebSecurityConfigurer的实现类。**在 Spring 应用上下文中,任何实现了 WebSecurityConfigurer 的 bean 都可以用来配置 Spring Security。**

> `@EnableWebSecurity` 可以启用任意 Web 应用的安全性功能,~~不过如果是使用Spring MVC 开发的,那么就应该考虑使用~~`@EnableWebMvcSecurity` 替代它——弃用了

****

可能我们希望指定 Web 安全的细节,这要通过**重载 WebSecurityConfigurerAdapter 的一个或多个方法来实现。我们通过重载 WebSecurityConfigurerAdapter的三个 configure()方法**,来配置Web安全性,这个过程中会使用**传递参数设置行为**

|方法|描述|
|:--:|:--:|
|configure(WebSecurity)|通过重载,配置 Spring Security 的 Filter链|
|configure(HttpSecurity)|通过重载,配置如何通过拦截器保护请求|
|configure(AuthenticationManagerBuilder)|通过重载,配置 user-detail 服务|

上面程序没有重写任何一个 configure() 方法,就是说明了为什么应用现在是**被锁定的**,尽管对于我们的需求来讲默认的 Filter 链,但是默认链的 configure(HttpSecurity) 等于如下:

```java
protected void configure(HttpSecurity http) throws Exception {
		logger.debug("Using default configure(HttpSecurity). If subclassed this will potentially override subclass configure(HttpSecurity).");

		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.formLogin().and()
			.httpBasic();
	}
```
这个默认配置指定了该如何保护 HTTP 请求,以及客户端认证用户的方案。`.authorizeRequests().anyRequest().authenticated()` 要求所有进入应用的 HTTP 请求都要进行认证。它也配置 Spring Security 支持基于表单登录和 Http Basic 方式的认证

其他两个方法默认没有规定,为了满足需求,还需要在添加一点配置。
- 配置用户存储
- 指定那些请求需要认证,那些请求不需要认证,以及所需要的权限
- 提供一个自定义的登录页面,代替原来简单的默认登录页

除了这些还希望基于安全限制,有选择性地在 Web 视图上显示特定的内容

# 选择查询用户详细服务(configure(AuthenticationManagerBuilder))

我们所需要的是用户存储,也就是用户名,密码以及其他信息存储的地方,在进行认证决策的时候,会对其进行检索

Spring Security 基于各种数据存储来认证用户
- 内存
- 关系型数据库
- LDAP
- 自定义用户存储

## 基于内存的用户存储
重载 configure() 方法,并以 AuthenticationManagerBuilder 作为传入参数,它有多个方法配置 Spring Security 对认证的支持。通过** inMemoryAuthentication() 方法,我们可以启用,配置并任意填充基于内存的用户存储**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()   //启动内存用户存储
                .withUser("user").password("password").roles("USER").and()
                .withUser("admin").password("password").roles("USER","ADMIN");
    }
}

```
其中添加两个用户,调用 withUser() 方法为内存存储添加新用户,这个方法返回 UserDetailsManagerConfigurer.UserDetailsBuilder,** 这个对象提供了多个进一步配置用户的方法,包括设置用户密码的 password() 方法以及授予一个或多个角色权限的roles() 方法。**
> 使用了设计模式的构造者模式

UserDetailsManagerConfigurer.UserDetailsBuilder 对象所有可以方法
![](http://ww1.sinaimg.cn/large/006rAlqhly1g3g4xfb8ftj30hl0hs40l.jpg)

|方法|描述|
|:--:|:--:|
|accountExpired(boolean)|定义帐号是否已经过期|
|accountLocked(boolean)|定义帐号是否已经锁定|
|and()|用来连接配置|
|authorities(GrantedAuthority)|授予某个用户一项或多项权限|
|authorities(List<? extends GrantedAuthority>)|授予某个用户一项或多项权限|
|authorities(String...)|授予某个用户一项或多项权限|
|credentialsExpired(boolean)|定义凭证是否已经过期|
|disabled(boolean)|定义帐号是否已被禁用|
|password(String)|定义用户的密码|
|roles(String...)|授予某个用户一项或多项角色|

## 基于数据库表进行认证
用户数据通常会存储在关系型数据库中,并通过 JDBC 进行访问。为了配置 Spring Security 使用以 JDBC 为支撑的用户存储，**使用 jdbcAuthentication()方法**

```java
@Autowired
    DataSource dataSource;
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .jdbcAuthentication()
                .dataSource(dataSource);
    }
```

### 重写默认的用户查询功能

默认能让一切运转起来,但是它对我们的数据库模式有一些要求。下面是内部的 SQL 查询语句

```java
public class JdbcUserDetailsManager extends org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl implements org.springframework.security.provisioning.UserDetailsManager, org.springframework.security.provisioning.GroupManager {
    public static final java.lang.String DEF_CREATE_USER_SQL = "insert into users (username, password, enabled) values (?,?,?)";
    public static final java.lang.String DEF_DELETE_USER_SQL = "delete from users where username = ?";
    public static final java.lang.String DEF_UPDATE_USER_SQL = "update users set password = ?, enabled = ? where username = ?";
    public static final java.lang.String DEF_INSERT_AUTHORITY_SQL = "insert into authorities (username, authority) values (?,?)";
    public static final java.lang.String DEF_DELETE_USER_AUTHORITIES_SQL = "delete from authorities where username = ?";
    // 省略...
}
```

如果数据库与上面所述不一致,可以通过自定义查询获取更多的控制权

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
                                                   throws Exception {
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery(
        "select username, password, true " +
        "from Spitter where username=?")
      .authoritiesByUsernameQuery(
        "select username, 'ROLE_USER' from Spitter where username=?");
}
```

### 使用转码后的密码

认证查询,它会预期用户密码存储在数据库之中.唯一问题如果密码明文存储,会很容易受到黑客的窃取。为了解决这个问题需要借助 **passwordEncoder() 方法指定一个密码转码器(encodee)**,思路是,将密码按照算法加密存入数据库,当用户输入密码,转吗进行比较,不需要将密码解密,避免泄漏

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
                                                   throws Exception {
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery(
        "select username, password, true " +
        "from Spitter where username=?")
      .authoritiesByUsernameQuery(
        "select username, 'ROLE_USER' from Spitter where username=?")
      .passwordEncoder(new StandardPasswordEncoder("53cr3t"));
}
```
passwordEncoder() 方法可以接受 Spring Security 中 passwordEncoder 接口的任意实现。Spring Security 加密模块:
1. BCryptPasswordEncoder
2. NoOpPasswordEncoder
3. StandardPasswordEncoder

## 基于 LDAP 进行认证

LDAP:轻量目录访问协议 ,我们可以使用 ldapAuthentication() 方法:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception{
    auth
        .ldapAuthentication()
        .userSearchFilter("(uid={0})")
        .groupSearchFilter("member={0}");
}
```

后两个方法为基础 LDAP 查询提供过滤条件,分别用于搜索用户和组,默认基础查询为空,**表明搜索会在 LDAP 层级结构的根开始**,我们可以通过指定基础来改变这个默认:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
                                                   throws Exception {
  auth
    .ldapAuthentication()
      .userSearchBase("ou=people")
      .userSearchFilter("(uid={0})")
      .groupSearchBase("ou=groups")
      .groupSearchFilter("member={0}");
}
```
> 声明用户应该在名为 people 的组织单元下搜索而不是根开始。而组应该在名为 groups 的组织单元下搜索。

### 配置密码比对

基于 LDAP 进行认证的默认策略进行绑定操作,直接通过 LDAP 服务器认证用户。另一种可选的方式是进行比对操作。**将输入的密码发送到 LDAP 目录上,并要求服务器将这个密码和用户的密码进行比对,因为在服务器完成,实际密码能保证私密**
passwordCompare()方法实现
```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
                                                   throws Exception {
  auth
    .ldapAuthentication()
      .userSearchBase("ou=people")
      .userSearchFilter("(uid={0})")
      .groupSearchBase("ou=groups")
      .groupSearchFilter("member={0}")
      .passwordCompare()
      .passwordEncoder(new Md5PasswordEncoder())
      .passwordAttribute("passcode");
}
```

默认情况下,在登录表单中提供的密码将会与用户的 LDAP 条目中的 userPassword 属性进行对比。如果密码被保存在不同的属性中,可以通过 passwordAttribute()方法声明密码属性名称。

> 上面可以看出 比对密码属性是 "passcode"，另外,我们还指定密码转码器,即使在 LDAP 服务器还是可能被黑客拦截,上面用了 MD5 加密。

### 引用远程的 LDAP 服务器

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
        throws Exception {
  auth
  .ldapAuthentication()
    .userSearchBase("ou=people")
    .userSearchFilter("(uid={0})")
    .groupSearchBase("ou=groups")
    .groupSearchFilter("member={0}")
    .contextSource()
      .url("ldap://habuma.com:389/dc=habuma,dc=com");
}
```

## 配置自定义的用户服务
需要自定义 UserDetailsService 接口:

```java
public interface UserDetailsService {
  UserDetails loadUserByUsername(String username)
                                     throws UsernameNotFoundException;
}
```
实现如下：

```java
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.
                                               SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.
                                                   UserDetailsService;
import org.springframework.security.core.userdetails.
                                            UsernameNotFoundException;
import spittr.Spitter;
import spittr.data.SpitterRepository;

public class SpitterUserService implements UserDetailsService {

  private final SpitterRepository spitterRepository;
//注入 SpitterRepository
  public SpitterUserService(SpitterRepository spitterRepository) {
     this.spitterRepository = spitterRepository;
  }

  @Override
  public UserDetails loadUserByUsername(String username)
      throws UsernameNotFoundException {
          // 查找 Spitter 对象
    Spitter spitter = spitterRepository.findByUsername(username);
     if (spitter != null) {
         // 创建权限列表
      List<GrantedAuthority> authorities =
          new ArrayList<GrantedAuthority>();
      authorities.add(new SimpleGrantedAuthority("ROLE_SPITTER"));
// 返回 user
      return new User(
           spitter.getUsername(),
          spitter.getPassword(),
          authorities);
    }
    throw new UsernameNotFoundException(
        "User '" + username + "' not found.");
  }

}

```

设置到安全配置中:
```java
@Autowired
SpitterRepository spitterRepository;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception{
    auth
    .UserDetailsService(new SpitterUserService(spitterRepository))
}
```

# 拦截请求(configure(HttpSecurity))
任何应用,都不是所有请求都保护,比如首页,需要开发,表单提交等则需要保护
**对每个请求进行西历度安全性控制的关键在于重载 configure(HttpSecurity)方法**
