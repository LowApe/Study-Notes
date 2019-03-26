## Object类
第一次很正式的把一个类拿出来分析,虽然很多人说应该具体项目遇到的问题再去研读源码,但是总觉得你先得知道有什么,问题能通过对源码的熟悉程度能被解决把..

![java 8 目录](http://ww1.sinaimg.cn/large/006rAlqhly1g1a65qztzlj308i0ad74n.jpg)

****

```java
private static native void registerNatives();
    static {
        registerNatives();
    }
```
问题:native 是什么? 这个方法是干什么的?<br>
 [Java native方法详解](https://blog.csdn.net/wangaiheng/article/details/78350687)

****
```java
public final native Class<?> getClass();
```
> 获取运行时类的 Class 对象

****

```java
 public native int hashCode();
```
## Object 中的hashCode() 与 System 中静态方法 identityHashCode(Object obj) 区别?

```java
Student student = new Student();
        String a= new String("123");
        String v= new String("123");
        System.out.println("o-a:"+a.hashCode());
        System.out.println("i-a:"+System.identityHashCode(a));
        System.out.println();

        System.out.println("o-student:"+student.hashCode());
        System.out.println("i-student:"+System.identityHashCode(student));
        System.out.println();
        System.out.println("o-v:"+v.hashCode());
        System.out.println("i-v:"+System.identityHashCode(v));

// 打印的值
// o-a:48690
// i-a:21685669
//
// o-student:2133927002
// i-student:2133927002
//
// o-v:48690
// i-v:1836019240

```
> Object 中的 hashCode() 是对象内存地址计算哈希值;String 类复写了 Object 的 hashCode() 所以它的 hashCode是[另一种计算]();System.identityHashCode()是对象内存地址计算哈希值;所以从上图可以看出,student 对象的 hashCode 相同

****

```java
public boolean equals(Object obj) {
       return (this == obj);
   }
```

## '==' 和 equals方法区别 ?

**==**:判断两个变量之间的值是否相等,变量可以分为`基本数据类型`,和`引用数据类型`
- 基本数据类型直接比较`值`
- 引用类型比较对应的引用内存的首`地址`

**equals**:判断两个对象的特征是否一样;这个存在与 equals 方法的复写方法的比较方式;

```java
int a=7;
        int b=7;
        Integer one=new Integer(7);
        Integer two=new Integer(7);

        System.out.println(a==b);

        System.out.println(one==two);
        System.out.println(one.hashCode());
        System.out.println(two.hashCode());
        System.out.println(one.equals(two));
        System.out.println(System.identityHashCode(one));
        System.out.println(System.identityHashCode(two));
// 打印
        // true
        // false
        // 7
        // 7
        // true
        // 21685669
        // 2133927002
```
> 稍微解读一下:上面的前两个打印,可以说明我们所说的基本数据类型和引用数据类型;我查看了一下源码,Integer 分装类的 hashCode 就是数值,equals 的比较方法是比较两个值;还有就是基本数据类型没有 equals 方法;

****

```java
protected native Object clone() throws CloneNotSupportedException;
```
可以看出这是一个 protected 修饰,也就是只能同一包下的子类内部能使用,不同包下必须复写方法,才能使用;
```java
public class Student implements Cloneable {
    String name;
    int age = 5;
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class  Main{
    public static void main(String[] args) {
        try {
               Student a= new Student();
               System.out.println(a.clone().equals(a));
               System.out.println(a.age);
               Student b = (Student) a.clone();
               System.out.println(b.age);
           } catch (CloneNotSupportedException e) {
               e.printStackTrace();
           }
    }
}
// 打印
// false
// 5
// 5
```
> 前面我一直没有复写方法,然后在思考 protected 不是继承的子类可以调用吗?由于 Object 默认所有类继承,所有我一直以为他们在同一个包下,其实他们不同包,所有 protected 限制了子类调用不同包下的父类方法;还有就是复制的类实际上开辟了新的堆空间;

## 浅克隆
定义:浅克隆仅仅复制了这个对象本身的成员变量;



```java

public class Teacher {
    public  String name;

    public Teacher(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}


public class Student implements Cloneable {
    Teacher teacher;
    int age = 5;
    public Student(Teacher teacher) {
           this.teacher = teacher;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class Main{
    public static void main(String[] args) {
        try {
               Teacher t=new Teacher("Tom");
               Student a= new Student(t);
               Student b= (Student) a.clone();
               System.out.println(System.identityHashCode(a)+"-"+System.identityHashCode(b));
                System.out.println(System.identityHashCode(a.teacher)+"-"+System.identityHashCode(b.teacher));
               System.out.println(a.teacher.name);
               System.out.println(b.teacher.name);
               t.setName("Jerry");
               System.out.println(a.teacher.name);
               System.out.println(b.teacher.name);
           } catch (CloneNotSupportedException e) {
               e.printStackTrace();
           }
    }
}
// 打印
// 21685669-2133927002
// 1836019240-1836019240
// Tom
// Tom
// Jerry
// Jerry
```
>  我先口述一下上面的过程(画图还没有找到好的工具);先看 main 方法,先创建一个 被克隆对象的成员变量 t,然后赋值;然后创建一个 a 对象,然后复制 a,当改变 t的值,被克隆和克隆对象成员都发生变化;

小总结:从hashcode 可以看出,克隆前后两个对象有自己独立的空间,但是赋值后的成员变量并没有克隆;

## 深克隆
定义:会复制这个对象和它所引用的对象的成员变量;需要在覆写 clone() 方法中添加对成员变量的克隆

```java
public class Teacher {
    public String name;

    public Teacher(String name) {
        this.name = name;
    }
    /**
   * 调用这个方法可以实现对此类的克隆
   * @return
   */
    @Override
    public Teacher clone()  {
        return new Teacher(getName());
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

public class Student implements Cloneable {
     Teacher teacher;
    private int age = 5;
    public Student(Teacher teacher) {
        this.teacher = teacher;
    }
    @Override
    protected Student clone() throws CloneNotSupportedException {
        Student student = (Student) super.clone();
        student.teacher = teacher.clone();
        return student;
    }
    public Teacher getTeacher() {
        return teacher;
    }

    public void setTeacher(Teacher teacher) {
        this.teacher = teacher;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

public class Main{
    public static void main(String[] args) {
        try {
                Teacher t=new Teacher("Tom");
                Student a= new Student(t);
                Student b=  a.clone();
                System.out.println(System.identityHashCode(a)+"-"+System.identityHashCode(b));
                System.out.println(System.identityHashCode(a.teacher)+"-"+System.identityHashCode(b.teacher));
                System.out.println(a.teacher.name);
                System.out.println(b.teacher.name);
                a.getTeacher().setName("Jerry");
                System.out.println(a.teacher.name);
                System.out.println(b.teacher.name);
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }
    }
}
// 打印
// 21685669-2133927002
// 1836019240-325040804
// Tom
// Tom
// Jerry
// Tom
```
> 实现深克隆,需要注意两个地方,一个是要在克隆类的成员变量实体类实现一个 public 的克隆方法,用来返回一个新的成员变量,这个用于改变克隆的成员变量;上面的hashcode 表明创建了新的空间;

- [clone() 的参考连接](https://www.kawabangga.com/posts/374)
****

```java
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```
> 得到 对象名+@+十六进制的hasCode;

****

> 一旦编译器碰到一个字串，后面跟随一个“+”，就一旦编译器碰到一个字串，后面跟随一个“+”，就

```java

class TestOne{

}
public class ToStringP {
    public static void main(String[] args) {
        System.out.println("显示"+(new TestOne()));
    }
}

//打印
// 显示eight.TestOne@1540e19d

```


```java
class TestOne{
    public String toString(){
       return "aaaa";
    }
}
public class ToStringP {
    public static void main(String[] args) {
        System.out.println("显示"+(new TestOne()));
    }
}

// 打印
// 显示aaaa
```
