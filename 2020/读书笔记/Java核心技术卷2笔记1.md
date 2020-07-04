# Java SE 8 的流库

## 迭代到流的相关概念

> 在处理集合时，我们通常会遍历它的元素，并在每个元素上执行某些操作，而使用流可以不必遍历整个代码去**过滤**和**计数操作**

流与集合区别：

- 使用流比迭代更具有**可读性**
- 流遵循的是**做什么而不是怎么做**
- 流并不存储元素。存储在底层集合，**按需生成**

书中示例代码：

```java
public class CountLongWords {
    public static void main(String[] args) throws IOException {
        String contents = new String(Files.readAllBytes(Paths.get("全局路径\longWords.txt")), StandardCharsets.UTF_8);
        List<String> words = Arrays.asList(contents.split("\\PL+"));
        // 集合调用 stream 方法转化为流
        long count = words.stream().filter(w -> w.length() > 8 ).count();
        System.out.println(count);
		// 并行的流
        long count2 = words.parallelStream().filter(w -> w.length() > 8 ).count();
        System.out.println(count2);
    }
}
```

操作流的典型流程：

1. **创建**一个流
2. 指定将初始流**转换**为其他流的中间操作，可能包含多个步骤
3. 终止操作，从而产生**结果**

## 流的创建

相关方法介绍：

- 数组使用 `Stream.of()`静态方法转换
- `Stream.of()`具有**可变长参数，可构建任意数量引元的流**
- `Array.stream(arry,from.to)`处理数组，从范围索引创建一个流
- `Stream.empty()`创建一个空的流
- `Stream.generate(Supplier<T>)`创建无限流的静态方法
- `Stream.iterate(final <T> speed,final UnaryOperation<T>)`创建无限流，反复将函数应用到之前到结果之上
- Java API 中有大量方法都可以产生流。如：`Pattern`

书中示例代码：

```java


public class CreateStreams {
    public static <T> void show(String title, Stream<T> stream) {
        final int SIZE = 10;
        List<T> firstElements = stream.limit(SIZE + 1).collect(Collectors.toList());
        System.out.println(title + ":");
        for (int i = 0; i < firstElements.size(); i++) {
            if (i > 0) System.out.println(",");
            if (i < SIZE) System.out.println(firstElements.get(i));
            else System.out.println("...");
        }
        System.out.println();
    }

    public static void main(String[] args) throws IOException {
        Path paths = Paths.get("全局路径/longWords.txt");
        String contents = new String(Files.readAllBytes(paths), StandardCharsets.UTF_8);
//      数组转换为流使用静态方法 of
        Stream<String> words = Stream.of(contents.split("\\PL+"));
//        show("words",words);

        Stream<String> song = Stream.of("A","B","C");
//        show("song",song);

        Stream<String> silence = Stream.empty();
//        show("silence",silence);
		// 无限输出，但规定最多打印前10个
        Stream<String> echos = Stream.generate(()->"echo"); 
//        show("echos",echos);

        Stream<Double> maths = Stream.generate(Math::random);
//        show("maths",maths);

        Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO,t->t.add(BigInteger.ONE));
//        show("integers",integers);

        Stream<String> wordsAnotherWay = Pattern.compile("\\PL+").splitAsStream(contents);
//        show("wordsAnotherWay",wordsAnotherWay);
        // 带资源的 try
        try(Stream<String> lines = Files.lines(paths,StandardCharsets.UTF_8)){
            show("lines",lines);
        }
    }
}
```



> - 引元：我理解为可变长的参数引用
> - `"\\PL+"`正则，非字母过滤



# 问题解决

# 相关链接