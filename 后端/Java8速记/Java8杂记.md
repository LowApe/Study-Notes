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
- 