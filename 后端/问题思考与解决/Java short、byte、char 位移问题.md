# short byte 和 char 会自动转换成 int 然后进行位移操作
```java
package util;

public class Displayment {
    public static void main(String[] args) {
        int i = -1;
        i >>>= 10;
        System.out.println(i); //print 4194303

        long l=-1;
        l>>>=10;
        System.out.println(l); // print 18014398509481983

        short s = -1;
        int ss = s >>> 10;
        System.out.println(s); // print -1
        System.out.println(ss); //print 4194303
    }
}

```
Q:上面的 short 为什么输出 -1
> 因为 short 直接转换成 int 所以右移10位就是 0000000000111111111111111111 (10 个0 22个 1) 为什么会输出 -1 ? 因为s是 short 类型所以 int -> short 阶段成 16 位 111111111111111111 => 加上符号位就是 -1
