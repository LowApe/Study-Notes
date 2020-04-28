## 4.1 面向对象程序设计概述

### 类

封装：暴露公共的实现方法和隐内部数据的实现方式,封装具有可扩展性即黑盒特性，不考虑内部实现，只要暴露接口不变，后期如何变更实现不会影响使用.

继承：

### 类之间的关系

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1gdyymayl0oj31g60a8jtx.jpg)

## 4.2 预定义类

例如：

1. Math
2. Date

### Java中的 LocalDate

```java
/* 静态方法提供时间 */
Localdate.now();// 获取当前时间 2020-04-19
Localdate.of(2020,2,2); // 提供年、月和日来构造特定日期时间 2020-02-02
int year = Localdate.getYear(); // 方法获取 Localdate 对象数据
int month = Localdate.getMonthValue();
int day = Localdate.getDayOfMonth();

```

### 更改器方法和访问方法器方法

```
LocalDate localVlaue = Localdate.of(2020,2,2);
localVlaue.plusDays(1000); // 并不会更改原本的日期
GregorianCalendar someDay = new GregorianCalendar(1997,2,2);
someDay.add(Calendar.DAY_OF_MONTH,1000); // 会更改 someDay 的数值
```

## 4.3 用户自定义类

构造器

私有参数

公共方法



### 隐式参数和显式参数

> 显式参数明显存在于方法的声明部分

```java
public void aMethod(String explicit){// explicit 为显式参数
	this.value = explicit * 2; // this 表示隐式参数
}
```

> ⚠️：不要编写返回引用可变对象的访问器方法，破坏封装性;如下代码：

```java
class Employee{
    private Date date;
}
Employee emp = new Employee();
Date d = emp.getDate();
// 此时 d 引用的同一个对象，d调用更改器方法就可以自动地改变emp 的私有状态
```

> 如果需要一个可变的引用，应该对其进行克隆，修改后 `getDate(){ return (Date)date.clone();}`

### final 实例域

被 final 修饰的变量初始化后不可修改



## 4.4 静态域和静态方法

私用参数可以通过使用 `static`修饰静态域，可以通过类直接引用而无需实例对象。

```java
class Sutent{
	private int static final id = 1;
}

class Test{
    public static void main(String args[]){
    	System.out.print(Student.id); // 无需实例 Student;
	}
}
```

### 静态方法

类似与 Math 的静态方法操作数值。

```java
class Student{
    private int id =1;
    
    public static int getId(){
        return id;
    }
}
class Test{
    public static void main(String args[]){
    	System.out.print(Student.getId()); // 无需实例 Student;
	}
}

```

### 工厂方法

> `LocalDate` 类`NumberFormat`类使用静态工厂方法

## 4.5 方法参数

> Java 方法参数是值传递，而非引用传递，因为传入对象在方法内部操作后对象改变值，如果是引入传递，相应的对象也应该改变

```java
class Money{
    public static void addMoney(Double x){
        x = x * 3;
    }
    
    public static void main(String args[]){
        Money money = new Money();
        Double num = new Double(10);
        money.addMoney(num);
        System.out.print(num);// 还是 10
	}
}
```

## 4.6 对象构造



### 重载

> Java 允许重载任何方法，而不是只有构造器方法.当不是构造器方法，返回类型不能作为重载标准

```java
// 下面是错误的❌
class Test{
     public int hello(){
        return 1;    
    }
    
    public String hello(){
        return "1";
    }
}
```

### 默认域初始化

自动默认：初始值 0，布尔类型 false，对象为 null，一般需要初始化

### 无参数的构造器

默认提供无参的构造器

### 显示域初始化

```java
class Example{
    private String name = "";
}
```

### 参数名

```java
public Employee(String name,String salary){ // 参数名称起解释作用
    this.name = name; // 参数名称与实例名称相同 通过this 引用
    this.salary = salary;
}
```



### 调用另一个构造器

如果构造器使用`this(...)` 则表示调用其他构造器，根据参数识别。

```java
class Example{
    public Example(int a){
        this("b");
    }
    public Example(String b){
    	System.out.print(b);
    }
    
}
```

> this 有两个作用：
>
> 1. 调用引用参数
> 2. 调用该类的构造方法

### 初始化块

```java
class Example{
	private int id;
	{
	   id = 1;
	}
}
```

> 初始化块在构造器前调用。顺序是
>
> 1. 数据域被初始化默认（0 false null）
> 2. 依次加载初始化域和初始化块
> 3. 构造器第一行调用了第二个构造器，则先执行第二个构造器
> 4. 

### 对象析构与finalize方法

Finalize 方法将在垃圾回收器清楚对象前调用，可以为任何对象实现finalize 方法，但是不建议，因为不知道什么时候对象被回收

## 4.7 包

> 注意编译器(javac)对带文件分隔符和扩展名`.java`的扩展名文件进行操作，而解释器加载类(带有.分隔符)



## 4.8 类路径



## 4.9 文档注释



## 4.10 类设计技巧

- 一定要保证数据私有
- 一定要保证数据初始化
- 不要再类中使用过多的基本数据类型(易理解、易于修改)
- 不是所有的域都需要访问器和更改器
- 需要将指责过多的类进行分解
- 类名和方法名要能够体现它们的职责
- 优先使用不可变的类