# Java 字符串转数字并格式化



```java
public class Test {
    public static void main(String[] args) {
        BigDecimal a = new BigDecimal("0.000");
        BigDecimal b = new BigDecimal("10.200");
        BigDecimal c = new BigDecimal("10.600");
        BigDecimal d = new BigDecimal("12.260");

        System.out.println(new DecimalFormat("#").format(a));
        System.out.println(new DecimalFormat("#").format(b));
        System.out.println(new DecimalFormat("0.##").format(a));
        System.out.println(new DecimalFormat("0.##").format(b));
        System.out.println(new DecimalFormat("0.##").format(c));
        System.out.println(new DecimalFormat("0.##").format(d));
        System.out.println(new DecimalFormat("#.##").format(a));
        System.out.println(new DecimalFormat("#.##").format(b));
        System.out.println(new DecimalFormat("#.##").format(c));
        System.out.println(new DecimalFormat("#.##").format(d));
  
```

