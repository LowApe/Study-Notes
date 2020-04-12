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



# 3.6 输入和输出

## 读取输入

```java
Scanner in = new Scanner(System.in);
String name = in.nextLine();// 获取输入行
String firstWord = in.next(); // 获取第一个单词 空格为分隔符
int intNumb = in.nextInt();
Double doubleString = in.nextDouble();

/* Java SE 6 特别引入 Console 类实现这个目的密码*/
Console con = System.console();
String userName = con.readLine("User name");
char[] password = con.readPassword("Password");
```

## 格式化输出

> 类似 C 格式化输出

## 文件输入和输出

```java
import java.nio.file.Paths;
Scanner in = new Scanner(Paths.get("myFile.txt"),"UTF-8"); // 读取文件
PrintWriter out = new PrintWriter("myFile.txt","UTF-8"); // 写入文件

String dir = System.getProperty("user.dir"); // 获取用户目录
```

> 如果使用命令行启动程序，可以通过 Shell 命令关联 `System.in`、`System.out`

```shell
java ClassName < file.txt > out.txt
```

# 3.9 大数值

如果整型和浮点型精度无法满足使用，可以使用 `java.math` 包中的两个很用的类`BigInteger`和`BigDeicmal`



```java
// 使用 valueOf 将普通数值转换为大数值
BigInteger bigInteger = BigInteger.valueOf(100);

```

> 注意无法使用 `+` 或者 `*` 来操作大数据，可以通过方法`add`和`multiply`

```java
BigInteger c = a.add(b);
BigInteger d = b.multiply(c);
```

# 3.10 数组

```java
/*数组两种声明*/
int[] a;
int a[];
```

## for each 循环

```java
for (variable : collection) statement
   
Arrays.toString(array);    // 不想循环打印数组中的数值
```

## 数组初始化以及匿名数组

```java
new int[]{1,21,2};
```

## 数组拷贝

通常通过两个变量之间的数组“拷贝”只不过是将引用指向相同的存储位置。如果需要创建一个新的数组，则需要

```java
int[] a = Arrays.copyOf(b,b.length);
```

## 命令行参数

```shell
java ClassName x1 x2 x3
```

所对应 Java 程序中

```java
class ClassName{
    public static void main(String args[]){
        System.out.println(Arrays.toString(args));
    }
}

[x1,x2,x3]
```

## 数组排序

```java
Arrays.sort(a)
```



## 多维数组

多维数组数组适合**表格**或者更多的复杂排列形式

```java
/*初始化*/
int[][] a;
a = new int[][];
int[][] b={
    {1,2,3},{2,3,4},{4,5,6}
}

/* 快速打印一个二维数组 */
Arrays.deepToString(a);
```

## 不规则数组

```java
int[][] odds = new int[5][];
```

