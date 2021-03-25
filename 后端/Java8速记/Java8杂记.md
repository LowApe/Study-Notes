# Java8 lambda 要点

- Lambda 表达式是一个匿名方法，将行为像数据一样进行传递

- 方法传入匿名内部类，转换为 Lambda 无需声明的类型参数，**通过 javac 根据程序上下文推断参数的类型**

- Lambda 的类型依赖于上下文环境

- Lambda 的几种表示形式

	- `() -> `
	- `event ->`
	- `()->{}`
	- `(x,y)-> x+y`
	- `(long x,long y) -> x+y`

- Java8 使用 Lambda 需要引用 `final`修饰的终态变量，虽然可以引用非终态，**要求Lambda表达式引用的是值，而不是变量**

	- Lambda 表达式称为闭包

- **函数接口是只有一个抽象方法的接口，用作 Lambda 表达式的类型**

- `java.util.function`下重要的函数接口

	- | 接口              | 参数  | 返回类型 | 示例                   |
		| ----------------- | ----- | -------- | ---------------------- |
		| Predicate<T>      | T     | boolean  | 这张唱片已经发行了吗   |
		| Consuemer<T>      | T     | void     | 输出一个值             |
		| Function<T,R>     | T     | R        | 获得 Artist 对象的名字 |
		| Supplier<T>       | None  | T        | 工厂方法               |
		| UnaryOperator<T>  | T     | T        | 逻辑(!)                |
		| BinaryOperator<T> | (T,T) | T        | 求两个数的乘积(*)      |

		



## 流

- 循环迭代，外部迭代(for循环)，内部迭代(Stream流迭代)
- Stream 是用函数式编程方式再集合类上进行复杂操作的工具
- 像`filter`只描述Strem，最终不产生新集合的方式叫作**惰性求值方法**
- 像`count`最终会从 Stream，产生值的方法叫作**及早求值方法**
- 如果返回值 Stearm 则是惰性求值方法，如果返回值是一个值或为空，则及早求值方法
- Stream 的链保证最后一个是及早求值，整个过程和**建造者模式**有共通之处

## 常用的流操作

- `collect(toList())`方法由Stream里的值生成一个列表，是一个及早求值操作
- `Stream.of()`初始化Stream流
- `map`：如果有一个函数可以将一种类型的值**转换**成另外一种类型，map 操作就可以使用该函数，将一个流中的值转换成一个新的流
	
- ![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gogvt71rq7j30f60543yh.jpg)
	
- `filter`:遍历数据并检查符合条件的元素
	
- ![image-20210312095159957](/Users/mac/Library/Application Support/typora-user-images/image-20210312095159957.png)
	
- `flatMap`：方法可用Stream**替换值**，然后将多个Stream连接成一个Stream

- `max`和`min`求最大值和最小值

	- ```java
		public void main(String[] args){
		    abc.stream
		        .min(Comparator.comparing(track -> tarck.getLength()))
		        .get()
		}
		```

- `reduce`操作可以实现从一组值中生成一个值。🤔



## 高阶函数

- 高阶函数是指接受另外一个函数作为参数或返回一个函数的函数
- Function
- ToLongFunction
- LongFunction

## 类库

> Java8 中的另一个变化是引入了**默认方法(default 关键字)**和**接口的静态方法**

- 为了减少性能开销，Stream 类的某些方法对基本类型和装箱类型做了区分。**在Java8 中，仅对整型、长整型和双浮点型做了特殊处理，提高系统效率**

- 如果方法返回类型为基本类型，则在基本类型前加 To

	- ![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gontdbi1dyj30ge04mdfu.jpg)

	

- 如果参数是基本类型，则不加前缀只需类型名即可

	- ![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gontaoiav6j30f803gweg.jpg)

- BinaryOperator 是一种特殊的BiFunction类型，**参数的类型和返回值的类型相同**。
- Lambda 表达式作为参数时，其类型由它的目标类型推导得出，推导过程遵循如下规则：
	- 如果只有一个可能的目标类型，由相应函数接口里的参数类型推导得出
	- 如果有多个可能的目标类型，由最具体的类型推导得出
	- 如果有多个可能的目标类型且最具体的类型不明确，则需人为指定类型
- `@FunctionInterface`注解，用作函数接口，并不是所有只含有一个方法的接口就可以是函数接口。该注解会强制检查一个接口是否符合函数接口的标准

## 默认方法

- `Iterable`接口中也新增了一个默认方法：forEach，该方法功能和 for 循环类似，

	- ```java
		default void forEach(Consumer<? super T> action) {
		    for (T t : this){
		        action.accept(t);
		    }
		}
		```

- 默认方法没有重写，实现类不需要实现默认方法
- 任何时候，一旦与类中定义的方法产生冲突，都要优先选择类中定义的方法 **类中定重写的方法优先级高于接口中定义的默认方法**。原因在于，与接口中定义的默认方法相比，类中重写的方法更具体
	- 对默认方法的工作原理，特别是在多重继承下的行为还没有把握，如下三条简单的定律可以帮助大家
	- 类胜于接口
	- 子类胜于父类
	- 如果两条都不适用，子类要么需要实现该方法，要么将该方法声明为抽象方法
- 静态类是防止工具的好方式
- 

> 权衡，接口和抽象类之间还是存在明显的区别。**接口允许多重继承，没有成员变量；抽象类可以继承成员变量，却不能多重继承**

## 高级集合类和收集器

### 方法引用

- Lambda 表达式经常调用参数，
- **方法引用，格式 [类名：方法名]**
- **构造方法引用，格式[类名：new]**
- **创建数组，格式String[]::new**

### 元素顺序

- 出现顺序的定义依赖于数据源和对流的操作，**在一个有序集合中创建一个流时，流中的元素就按出现顺序排序**
- 流的目的不仅是在集合类之间做转换，而且同时提供了一组处理数据的通用操作。有些集合本身是无序的，但这些操作后时会产生顺序
- 一些操作在有序的流上开销更大，调用`unorderd`方法消除这种顺序就能解决该问题。**大多数操作都是在有序流上效率更高，比如 fliter、map和reduce**

### 使用收集器

- 一种通用的、从流生成复杂值的结构。只要将它传给`collect 方法`，所有的流都可以使用它,`Collectors`

	- `toList()`
	- `toSet()`
	- `toMap()`
	- `toCollection()` 定制的集合收集元素`stream.collect(toCollection(TreeSet::new))`

- **可以利用收集器让流生成一个值。**

- `collect(MaxBy(comparing(xxx)))`

- `collect(averagingInt(Function))`

- **另外一个常用的流操作是将其分解成两个集合**

-  `partitioningBy`它接收一个流，并将其分成两个部分。他使用 `Predicate` 对象判断一个元素应该属于哪个部分，并根据布尔值返回一个 Map 到列表

	- ```java
		public Map<Boolean,List<Object>> bansAndSolo(Stream<Artist> artists){
		    return artists.collect(partitioningBy(artist -> artist.isSolo()));
		}
		```

- **数据分组是一种更自然的分割数据操作**

	- ```java
		public Map<Artist,List<Album>> albumsByArtist(Stream<Alum> albums){
		    return albums.collect(groupingBy(album) -> album.getMainMusicaian()));
		}
		```

- `Collectors.joining()` **收集流中的数据都是为了在最后生成一个字符串**



### 组合收集器

- 先分组后计数

	- ```java
		public Map<Artist,Long> numberOfAlbums(Stream<Alum> alblums){
		    return albums.collect(Collectors.groupingBy(album -> album.getMainMusician(),Collectors.counting()))
		}
		```

	- 第二个方法可以将分组的流，进行二次处理输出值

### 重构和定制收集器

> 尽管在常用流操作里，Java 内置的收集器已经相当好用，但收集器框架本身是极其通用的。可以定制我们自己的收集器

**定义字符串收集器**

```java
public class StringCollector implements Collector<String,StringCombiner,String>{
    
}
```



- Collector<T,A,R> 收集器接口，三个泛型

	- 代收集元素的类型
	- 累加器的类型
	- 最终结果的类型

	> 一个收集器由四部分组成。首先是一个 Supplier，这是一个工厂方法，用来创建容器

	```java
	public Supplier<T> supplier(){
	    return ()->new T();
	}
	```

	

## 数据并行化

- 使用并行化流还是串行化，**基于流的代码是否比串行化运行更快**，通常数据量少的情况使用普通的流，如果数据量大使用并行化流效率较大
- 输入流的大小并不是决定并行化是否会带来速度提升的唯一元素，**性能还会受到编写代码的方式和核的数量的影响**
- 影响并行流性能的主要因素：
	- 数据大小
	- 源数据结构
	- 装箱
	- 核的数量
	- 单元处理开销
- 在要对流求值时，不能同时处于两种模式，要么是并行的，要么是串行的。如果同时调用了 `parallel`和`sequential`方法，最后调用的那个方法起效
- 我们可以根据性能的好坏，将核心类库提供的通用数据结构分为以下3组
	- 性能好：ArrayList、数组或 IntStream.range，这些数据结构支持随机读取，也就是说它们能轻而易举地被任务分解
	- 性能一般：HashSet、TreeSet
	- 性能差：LinkedList

- 数据并行化是把工作拆分，同时在多核CPU上执行的方式
- 如果使用流编写代码，可通过调用 parallel 或者 parallelStream 方法实现数据并行化操作

## 并行化数组操作

这些操作都在工具类`Arrays`中

| 方法名         | 操作                           |
| -------------- | ------------------------------ |
| parallelPrefix | 任意给定一个函数，计算数组的和 |
| parallelSetAll | 使用 Lambda 表达式更新数组元素 |
| parallelSort   | 并行化对数组元素排序           |

```java
/**
* for 循环初始化数组
*/
public static double[] imperativeInitilize(int size){
    double[] values = new double[size];
    for(int i = 0; i < values.length;i++){
        values[i] = i;
    }
    return values;
}
/**
* 使用并行化数组操作初始化数组
*/
public static double[] parallelInitialize(int size){
    double[] values = new double[size];
    Arrays.parallelSetAll(values,i -> i);
    return values;
}
```

- `parallelPrefix`操作擅长对时间序列数据做累加，它会更新一个数组，将每一个元素替换为当前元素和其前驱元素的和，这里的"和"是一个宽泛的概念，它不必是加法，可以是任意一个`Binaryoperator`
- 