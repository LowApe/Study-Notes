# Feign报错'xx.FeignClientSpecification', defined in null, could not be registered.

多个 `@FeignClient(“xxx”)`  相同的服务名报错

```properties
spring.main.allow-bean-definition-overriding: true
```

其他解决方法:

```java
@FeignClient(name="common-service", contextId = "example")
```

通过添加 contextId 来区分不同 Feign,亲测有效，需要不同的 contextId 名称

# 相关连接

- [Feign报错'xx.FeignClientSpecification', defined in null, could not be registered.](https://www.jianshu.com/p/16a526379e8f)
- [Feign 报错2](https://blog.csdn.net/u012211603/article/details/84312709)

