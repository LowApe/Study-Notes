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