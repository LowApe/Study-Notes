





# 空指针终结者：Optional<>

Optional 第一种使用技巧是在自动注入对象时使用，**当对象为空则不会被注入进来**

```java
@Service
public class DefaultUserService implements UserService{
    @Autowired
    private Optional<UserDao> userDao;
    
    public User findUserByUserName(String userName){
        if(userDao.isPresent()){
            userDao.get().findUserByUserName(userName);
        }
        return null;
    }
}
```

另一个使用 Optional 的地方时 Spring MVC 框架。使用 Optional 表示请求参数可选

```java
@RequestMapping("/user")
public User getUser(String id,Optional<String> userName){}
```

# 核心容器的增强

## 泛型依赖注入

```java
public abstract class BaseService<M extends serializable>{
    @Autowired
    protected BaseDao<M> dao;
}
@Service
public class UserService extends BaseService<User> {
    
}

@Service
public class ViewSpaceService extends BaseService<ViewSpace>{
    
}
```

## Map 依赖注入

```java
@Autowired
private Map<String,BaseService> map;
```

将 BaseService 类型注入 map 中，**key是 Bean 的名字：value 是所实现了 BasesServicede1 Bean**

## @Lazy延迟依赖注入

```Java
@Lazy
@Service
public class UserService extends BaseService<User>{
    
}
```

也可以把`@Lazy` 放在`@Autowired`之上，即依赖注入也是延时的，当调用 userService 时才会注入。同样适用于`@Bean`

## List 注入

```java
@Autowired
private List<BaseService> list;
```

**这样会注入所有事实现了 BaseService 的 Bean**，但顺序是不确定的。可以使用`@Order`或`Ordered`接口来实现排序

```java
@Order(value = 1)
@Service
public class UserService extends BaseService<User>{
    
}
```

