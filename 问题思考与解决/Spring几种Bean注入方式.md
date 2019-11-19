# Spring 几种Bean注入方式

常见的Bean注入注解有

- @Resource
- @Inject
- @Autowired

@Autowired 是 Spring 提供的注解，其他几个都是 JDK 本身内建的注释，Spring 对这些注释也进行了支持

## 三者之间的区别

1. `@Autowired` 有个 required 属性，配置为 false，这种情况下如果没有找到对应的 bean 是**不会抛异常**的。`@Inject` 和 `@Resource` 没有提供对应的配置，所有必须找到

2. `@Autowired` 和 `@Inject` 基本是一样的，因为两者都是使用 `AutowiredAnnotationBeanPostProcessor` 来处理依赖注入。但是 `@Resource`是个例外，它使用的是 CommonAnnotationBeanPostProcessor 来处理依赖注入。当然，两者都是 BeanPostProcessor



| @Autowird 和 @Inject                                         | @Resource                                                |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| 默认自动注入根据**类型**                                     | 默认自动注入根据**名称**                                 |
| 通过 @Qualifier 显式指定 名称                                | 通过@Qualifier 显式指定 名称                             |
| 如果类型查找失败，找不到或者多个实现<br />则退化为按**字段名称**查找 | 如果qualifier name名称查找失败，会退化为根据类型字段名称 |
| 可以在构造方法<br />setter方法注入                           |                                                          |

