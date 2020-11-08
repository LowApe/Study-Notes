# Mokcit 打桩错误



```java
    @Autowired
    private ItemApplicationService itemApplicationService;

    @MockBean
    private CategoryServiceFeign categoryServiceFeign;

    @MockBean
    private ItemServiceFeign itemServiceFeign;
```

错误信息

```java
org.mockito.exceptions.misusing.MissingMethodInvocationException: 
when() requires an argument which has to be 'a method call on a mock'.
For example:
    when(mock.getArticles()).thenReturn(articles);

Also, this error might show up because:
1. you stub either of: final/private/equals()/hashCode() methods.
   Those methods *cannot* be stubbed/verified.
   Mocking methods declared on non-public parent classes is not supported.
2. inside when() you don't call method on mock but on some other object.
```

问题解决

```java
// 使用 @MockBean 模拟被打桩的服务 而不是 @AutoWire
```

