# Mocktio

>  综合这几天查看的相关内容，初步总结一下，所谓 mock 数据，就是当我们在测试某一个功能方法时，该方法完成的功能可能依赖于一个或多个其他环境(Redis、Zookeeper、Http、DB等)，我们通过mock其他环境，也就是模拟其他环境行为的发生，实际测试过程中，不通过方法调用，而是通过预期设定的结果，该方法拿取其他环境预期结果进行测试。

## 例子

例子选用一个博主的例子。

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gg6vugsmkhj31280d4tco.jpg)

> 图更能解释上面我说的，当我们测试一个需要调用其他环境的模块时，如何保证当前测试模块的独立性,这里通过 mock 模块 B 和模块 C 来隔离模块D和模块E的影响

假设我们有下面这个类，`isAdmin(String user)` 这个方法是判断用户是否`admin`，但是被测试的方法内使用了 `Redis`进行获取用户名称，依旧是当前模块A里依赖于另一个模块B。

```csharp
public class RedisDemo {
    private Jedis jedis;

    public void setUp() {
        jedis = new Jedis("127.0.0.1", 6379);
        jedis.connect();
    }

    public boolean isAdmin(String user) {
        String ret = jedis.get("name");
        if (ret.equals(user)) {
            return true;
        }
        return false;
    }

    public static void main(String[] args) {
        RedisDemo demo = new RedisDemo();
        demo.setUp();
        boolean isAdmin = demo.isAdmin("test");
        System.out.println(isAdmin);
    }
}

```

单元测试：

```java
@Test
    public void test() {
        RedisDemo redisDemo = new RedisDemo();
        redisDemo.setUp("127.0.0.1", 6379);
        Assert.assertSame(true, redisDemo.isAdmin("admin"));
    }
```

单元测试看似我们完成了 `isAdmin`方法的测试，但是这个单元测试还有 `Redis`这个环境的影响，不能保证不收到外界的影响，使用 Mock 也是符合阿里Java开发单元测试强调的第4点:

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gg6vwlug3dj31ii0aqtcs.jpg)

我们可以使用模拟 `Redis` 的环境来隔离其他的依赖

## 使用 Mockito 进行测试

Spring Boot 只要导入`test`起步依赖就包含了Mockito，如果是Spring 使用下面的依赖：

```xml
<dependencies>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-all</artifactId>
            <version>1.8.5</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
</dependencies>
```

编写单元测试：

```java
public class RedisTest {
    @Rule
    public ExpectedException exception = ExpectedException.none(); // 不允许有异常
    
    @Test
    public void mockTest() throws Exception {
        // mock Jedis 对象
        Jedis mockJedis = Mockito.mock(Jedis.class);
        // 指定预期的结果
        Mockito.when(mockJedis.get("name")).thenReturn("admin");
		// 测试目标
        RedisDemo redisDemo = new RedisDemo();
        // 反射获取RedisDemoe 私有数据域  
        Field daoField = redisDemo.getClass().getDeclaredField("jedis");
        daoField.setAccessible(true);
        daoField.set(redisDemo, mockJedis);//??? 这里
		// 断言结果
        Assert.assertSame(true, redisDemo.isAdmin("admin"));
    }
}
```

当程序调用jedis的方法时，并不会真正触发jedis的方法，而是返回一个设定的值。



## Mockito 注解

### @MockBean

> 将 Bean 添加到 Spring 的上下文环境中,一旦加入，当测试执行到被添加该注解的服务，如果没有进行打桩，则默认为空
>
> 我的理解：使用这个注解的一般是外部系统调用：如 Redis、Mq

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gimouva4hlj31f00k4whs.jpg)

### @Mock

> 不添加到 Spring 的上下文环境中

# 认识单元测试的打桩

因为第一次理解这个概念，英文 stubbing，桩是用来代替**关联代码或者未实现的代码**。如果函数 B 用 B1 来代替，那么，B称为原函数，B1 称为桩函数。**打桩就说编写或生成桩代码**。 下面是官网的示例(stub method calls ):

```java
// you can mock concrete classes, not only interfaces
// 需要 mock 的类而不是接口
LinkedList mockedList = mock(LinkedList.class);

// stubbing appears before the actual execution
// 打桩出现在实际执行之前
when(mockedList.get(0)).thenReturn("first");

// the following prints "first"
// 返回 first 并没有执行方法
System.out.println(mockedList.get(0));

// the following prints "null" because get(999) was not stubbed
// 因为 get(999) 并没有打桩 打印 null
System.out.println(mockedList.get(999));
```

## 打桩的目的

![image-20200627161117685](/Users/mac/Library/Application Support/typora-user-images/image-20200627161117685.png)

- 隔离
- 补齐
- 控制

### 隔离

> 隔离是指将测试任务从产品分类出来，使之能够独立变异、连接，并**独立运行**，隔离的基本方法就是打桩，将测试任务之外与测试任务相关的代码，用桩来代替，从而实现**分离测试任务**。例如函数A调用了函数B，函数B又调用了函数C和D，如果函数B用桩来代替，函数A就完全隔断与函数C和D的关系

### 补齐

> 补齐是用桩来代替**为实现的代码**，例如，函数A调用了函数B，而函数B由其他程序员编写，且未实现，那么，可以用桩来代替函数B，使函数A能够运行。**补齐在并发开发中很常用**

### 控制

> 在测试时，人为设定相关代码的行为，使之符合测试需求

```java
public int B(){
	return Math.random();
}

public int A(){
    int let = B();
    if(let == xx){
        // do somthing
    }else if(let == xx){
        // do somthing
    }
    // 更多...
}
```

![image-20200627162328785](/Users/mac/Library/Application Support/typora-user-images/image-20200627162328785.png)

函数 B 返回随机数，或者返回网络状态，或者返回环境温度等等，当调用实际代码时，函数 A 很难测试，这时可以用桩函数 B1 来代替B，返回测试所需要的数据。**一个桩函数，可能既有控制功能，又具有隔离或者补齐功能**



> 桩代码并不是 Mock 代码

# 单元测试相关

其实单元测试类型从小到大一般有以下：

- 底层数据库调用的测试

- 端到端测试，一般是模拟API接口请求进行测试
- 集成测试，模拟用户操作的测试

这片文章所使用的打桩技术普遍存在于中间层，也就是 API 接口以及业务逻辑服务层

# 相关链接

- [Mockito入门及配合Junit进行单元测试](https://www.jianshu.com/p/46afe8c54d34)
- [Mockito官方网站](https://site.mockito.org/)
- [Mockito官方Javadoc](https://www.javadoc.io/doc/org.mockito/mockito-core/2.11.0/org/mockito/Mockito.html)
- [认识单元测试中的打桩](https://blog.csdn.net/wangwencong/article/details/8189778)
- [Mockito 单元测试打桩神器][https://www.bookstack.cn/read/hands-on-microservices/spring-boot-sb-mockito.md]