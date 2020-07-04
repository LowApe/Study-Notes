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



> - 引元：我理解为可变长的参数引用，其中元的话，理解成传入参数的类型到返回类型的一个元素
> - `"\\PL+"`正则，非字母过滤

## filter map 和 flatMap 方法

**流的转换会产生一个新的流，它的元素派生自另一个流中的元素。**

用到的转换流方法

- `Stream<T> filter(Predication<? super T> predicate)` 产生一个流，包含流中满足**断言条件**的元素
- `<R> Stream<R> map(Function<? super T,? extends R> mapper)` 产生流，包含 mapper 应用于当前流中所有元素所产生的结果
- `<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R> mapper)`产生流，通过将 mapper 应用于当前流所有元素所产生的结果连接到一起而获得。**每个结果都是一个流**

> Mapper 可以理解为方法的执行体



书上代码示例：

```java
public class StreamTransfer {
    public static void main(String[] args) {
        List<String> wordList = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            wordList.add("word"+i);
        }

        // 使用 filter 转换流
        Stream<String> stream = wordList.stream().filter(w-> w.length() >5);
        System.out.println(stream.collect(Collectors.toList()));

        // 使用 map 转换流中的值 Function<? super T, ? extends R> mapper
        Stream<String> stream1 = wordList.stream().map(String::toUpperCase);
        System.out.println(stream1.collect(Collectors.toList()));

        Stream<Stream<String>> stream2 = wordList.stream().map(w->letters(w));
        System.out.println(stream2.collect(Collectors.toList()));
        
        // 使用 flatMap 摊平
        Stream<String> stream3 = wordList.stream().flatMap(StreamTransfer::letters);
        System.out.println(stream3.collect(Collectors.toList()));
    }

    public static Stream<String> letters(String s){
        List<String> result = new ArrayList<>();
        for (int i = 0; i < s.length(); i++) {
            result.add(s.substring(i,i+1));
        }
        return result.stream();
    }
}

```

## 抽取子流和连接流

- `limit`方法裁剪流，该方法保留前 n 个
- `skip`该方法跳过前 n 个
- `Stream.concat(Stream<? extends T> a,Stream<? extends T> b)`连接两个流，**第一个流不应该是无限的**

书中代码示例:

```java
public class ChildConcatStream {
    public static void main(String[] args) {
        List<String> wordList = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            wordList.add("word"+i);
        }
        // 抽取前 n 个流
        Stream<String> stream= wordList.stream().limit(5);
//        System.out.println(stream.collect(Collectors.toList()));

        // 跳过前 n 个流
        Stream<String> stream1 = wordList.stream().skip(10);
//        System.out.println(stream1.collect(Collectors.toList()));

        // 连接的流如果引用过，就不能在使用了
        Stream<String> concatStream = Stream.concat(stream,stream1);
        System.out.println(concatStream.collect(Collectors.toList()));
    }
}

```

> **流在使用过后将不能再使用了，这也是为什么我需要把上两个打印的流注释掉**否则回报**流已关闭**的异常

```verilog
Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
	at java.util.stream.AbstractPipeline.spliterator(AbstractPipeline.java:344)
	at java.util.stream.Stream.concat(Stream.java:1080)
	at two.stream.ChildConcatStream.main(ChildConcatStream.java:27)
```



## 其他的流转换

- `Stream<T> distinct()`产生一个所有不同元素的流
- `Stream<T> sorted(Comparetor<? super T> comparetor)`产生一个流，它的元素是当前流中所有元素按照顺序排序。方法是**实现了 Comparable 的类实例**
- `Stream<T> peek(Consumer<? super T> action)` 产生一个流，与**当前流元素相同**，获取每个元素，**都会传递个 action**去做操作



书上的代码示例：

```java
public class OtherStream {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            if(i==1 || i==2 || i==3){
                list.add("wordsSame");
                continue;
            }
            list.add("words"+i);

        }
        // distinct 产生一个去重复的流
        Stream<String> stream = list.stream().distinct();
        System.out.println(stream.collect(Collectors.toList()));

        // 排序
        Stream<String> stream1 = list.stream().sorted(Comparator.comparing(String::hashCode).reversed());
        System.out.println(stream1.collect(Collectors.toList()));

        // peck 每次获取元素调用一个函数
        Stream<Double> stream2 = Stream.iterate(1.0,s-> s * 2).peek(e-> System.out.println("log"+e)).limit(20);
        System.out.println(stream2.collect(Collectors.toList()));
    }
}

```

## 简单简约(终结操作)

简约是一种**终结操作**，之前的`count`方法返回流中的元素数量

- `max` 和 `min` 简单简约返回`Optional<T>`的值。两个方法参数**比较器定义的排序规则**
- `Optional<T> findFirst()`
- `Optional<T> findAny()` ~~书上说分别返回第一个和任意一个~~，但是仅仅返回一个而且是第一。如果`null`产生一个空的 Optional 对象。这些操作都是终结操作
- `boolean anyMatch(Predicate<? super T> predicate)`
- `allMatch` 
- `noneMatch`

> 关于Optional 下一节会介绍

```java
public class ResultStream {
    public static void main(String[] args) {
        List<String> wordList = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            wordList.add("word"+i);
        }
        Optional<String> result = wordList.stream().max(String::compareToIgnoreCase);
        System.out.println(result);

        // findFirst() 结果第一个
        Optional<String> result2 = wordList.stream().filter(w-> w.startsWith("w")).findFirst();
        System.out.println(result2);

        // findAny 返回任意一个
        Optional<String> result3 = wordList.stream().findAny();
        System.out.println(result3);

        // 如果只是判断是否存在 可以使用 anyMatch 替换 filter
        boolean bl = wordList.stream().anyMatch(w-> w.startsWith("w"));
        System.out.println(bl);

        // allMatch
        boolean bl2 = wordList.stream().allMatch(w-> w.startsWith("w"));
        System.out.println(bl2);

        // noneMatch
        boolean bl3 = wordList.stream().noneMatch(w-> w.startsWith("w"));
        System.out.println(bl3);
    }
}

```



# 问题解决

# 相关链接

- [百度单子论]([https://baike.baidu.com/item/%E5%8D%95%E5%AD%90%E8%AE%BA/1429249?fr=aladdin](https://baike.baidu.com/item/单子论/1429249?fr=aladdin))