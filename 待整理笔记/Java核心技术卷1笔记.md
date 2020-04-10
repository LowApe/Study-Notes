> 总结记录容易遗忘或者常用的笔记



# 3.5 运算符

## 数学函数与常量

```java
double x = 4;
double y = Math.sqrt(x); // 数学平方根

double y = Math.pow(x,2); // 数学幂指数

/* Math 常用三角函数*/
Math.sin
Math.cos
Math.tan
Math.atan
Math.atan2
    
/* Math 的用于表示π，e */
Math.PI
Math.E
```



## 数值类型之间的转换



![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1gdp16jabt1j30xg0k84b4.jpg)



## 位运算符

&(and)  |(or) ^(xor) ~(not)



1. 同时满足 1&1=1 
2. 满足一个 1&0=1
3. 异或 1&1=0
4. 非

> `&` `|` 也可以判断布尔类型，但是不同于 `&&` 和`||` 会“短路”
>
> `>>` `<<`运算符将 位模式右移或着左移，其中 `>>>` 无符号运算符会用 0 填充高位.
>
> 位移需要将整数转换为 32 位(32是int 的 long，需要 64位)的二进制

## 括号与运算符级别



```java
a+=b+=c // 等价于
a+=(b+=c)
```



# 3.6 字符串

> 从概念上 Java 字符串是 Unicode 字符序列。Java 没有内置字符串类型，预定义类字符串 String

## 子串

```java
String e = "content";
String s = e.substring(0,3); // 使用 substring 获取子串
```

## 拼接

```java
String a = "hello" + "world"; // + 拼接字符串
```

> String 是不可变字符串，有一个优点：编译器可以让字符串共享,只有常量是共享的

```java
char gretting[] = "Hello";
```

> Java 字符串类似于 char* 指针

## 检测字符串是否相等

```java
String s = "hello";
s.equals("hello");
if(s.compareTo("hello") == 0);
"hello".equalsIgnoreCase(s);// 忽略大小写
```

> 一定不要使用 `==`判断字符串是否相等，它是判断两个字符串是否存放在同一个位置。



## 构建字符串

如果需要小段字符串

```java
StringBuilder builder = new StringBuilder();
builder.append("hello");
builder.append("abce");
```



