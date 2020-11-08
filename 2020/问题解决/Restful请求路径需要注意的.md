# Restful 请求路径写法



> 场景: 在 Spring Boot 中写请求路径的时候，当启动项目，路径所映射的的方式不同

```java
   @Override
    @GetMapping(value = "/queryOne",produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public ResultVo<CustomerResVo> queryOneList(@RequestParam Long id) {
        CustomerResVo resVo = customerService.queryOneCustomer(id);
        return ResultVo.ok(resVo);
    }
```

这种写法所映射出来的`url`路径是

```http
http://xxx:port/queryOne?id=xxxxx
```

> 



看一下不同的：

```java
    @Override
    @GetMapping(value = "/queryOne/{id}",produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public ResultVo<CustomerResVo> queryOneList(@PathVariable("id") Long id) {
        CustomerResVo resVo = customerService.queryOneCustomer(id);
        return ResultVo.ok(resVo);
    }
```

```http
http://xxx:port/queryOne/xxxxx
```



# 注意问题



> 坚决不要使用这种方式拼接 url 发请求，会造成请求数据传到后端，数据不一致，比如 我请求 20、10，多次调用接口，使用20，就可能出现传到后端变成10

应该是注解里面要加对应的参数



## 使用 Get 请求字符串拼接

尽量使用 POST 把请求的参数放到body里，而不是路径上

`@RequestParam`注解是 请求变量名称 `？ids=xxxx`