# JavaScript语言核心

# 2 词法结构

> 本节介绍 JavaScript 的词法结构

## 1 字符集

JavaScript 程序是用 **Unicode 字符集**编写的。Unicode 是 ASCII 和 Latin-1 的超集，并支持地球上几乎所有在用的语言。

- JavaScript 是区分大小写的语言
- HTML 并不区分大小写（XHTML区分大小写）
- JS会忽略程序中标识之间的空格🤔
- JS 识别
	- 普通的空格符(\u0020)
	- 水平制表符(\u0009)
	- 垂直制表符
	- 换页符
	- 不中断空白
	- 字节序表记
	- 以及在Unicode中所有 Zs 类别的字符🤔
- 
- 

> Unicode 这块好多看不懂说明，总的来说就是JS能识别特殊的编码表达一些特殊的行为

## 2 注释

行尾`//`之后文本会被忽视 `/**/`当作注释



## 3 直接量

> 程序直接使用的数据值

- 12//数字
- 1.2//小数
- “hello world” //字符串文本
- “hi” //另一个字符串
- true // 布尔值
- false // 另一个布尔值
- /javascript/gi // 正则表达式直接量
- null // 空

## 4 标示符和保留字

> JS中，标示符用对变量和函数进行命名，或者用做 JavaScript 代码中某些循环语句中的跳转位置的标记。
>
> **JS标识符必须以字母、下划线或美元符开始**
>
> 📒：我自己的理解就是变量或者函数的名称都可以说是标识符，然后为了保证标识符不重复，可以以字母、开始，区别数字
>
> JS 保留的一些标识符为保留字，保留字不能做普通的标识符（也就是这些的名称）

|        |            |          |          |
| ------ | ---------- | -------- | -------- |
| break  | delete     | function | return   |
| typeof | case       | do       | if       |
| switch | var        | catch    | else     |
| in     | this       | void     | continue |
| false  | instanceof | ...      |          |

JS同样保留了一些关键字，在未来使用.ES5保留的

`class const enum export extends import super`

ES3 保留了所有 Java 的关键字

JavaScript 预定义了很多全局变量或函数，应当避免把它们当作名字

`arguments encodeURI Number ...`

## 5 可选的分号

分号结尾

# 3 类型、值和变量

JS 的数据类型分为两类：

- 原始类型(primitive type)
- 对象类型(object type)

特殊原始值：null(空)和 undefined(未定义)

## 1 数字

JS 不区分**整数值**和**浮点数**值。 JS 中的所有数字均用浮点数值表示。JS 采用 IEEE 754 标准定义 64 位浮点格式表示数字。整数范围(-2^53 ~2^53),包括边界值

- 整型直接量
	- JS 能识别除了十进制之外的十六进制(0x)
- 浮点型直接量
	- 3.14
	- .3333333333
	- 6.02e23
	- 1.34344E-32
- JS支持基本的算术运算符(+-*/)，更复杂的使用 Math对象
	- Math.pow(2,53)  53次幂
	- Math.roud(.6）四舍五入
	- Math.ceil(.6) 向上取整
	- Math.floor(.6) 向下取整
	- Math.abs(-5) 求绝对值
	- Math.max(x,y,z) 返回最大值
	- Math.min(x,y,z) 返回最小值
	- Math.random() 随机生成一个大于等于 0 小于1.0 的伪随机数
	- Math.PI 圆周率
	- ...
- JS 算数存在溢出、下溢或被零整除时不会报错。数字溢出，结果为一个特殊的无穷大(+_infinity),0/0为 NaN
- 二进制浮点数和四舍五入错误
	- var x=.3-.2;var y=.2-.1 x==y // false
	- x 和 y 非常接近彼此单并不相等
- 日期和时间,JS 包括 Date() 构造函数，用来创建表示日期和时间的对象
	- var then = new Date(2011,0,1)
	- then.getFullYear()
	- then.getMonth()
	- ...

## 2 文本

字符串直接量

- ""

- 'testing'

- "3.14"

- 'name="myform"'

- ES5 字符串直接量可以**拆分成数行，每行必须必须反斜杠\\结束**

- **转义字符** \u 表示 4 个十六进制数指定的任意 Unicode 字符 \u03c0 表示字符π

	- | 转义字符 | 含义               |
		| -------- | ------------------ |
		| \o       | Nul字符(\u0000)    |
		| \b       | 退格符(\u0008)     |
		| \t       | 水平制表符(\u0009) |
		| ...      |                    |

- 字符串的使用
	- '+'号连接
	- .length 长度
	- .charAt(0) 第一个字符，[0]也可以访问第一个字符
	- ...
- **模式匹配**
	- RegExp 构造函数，创建表示文本匹配模式的对象(正则表达式)
	- 例如 `/^HTML/`以 HTML 开头的字符串
	- `/[1-9][0-9]*/`匹配一个非零数字，后面是人一个数字
	- `/\bjavascript\b/i`匹配单词 javascirpt，i忽略大小写
	- ...
	- RegExp 对象定义了很多有用的方法，字符串同样具有可以接受 RegExp 参数的方法
		- search
		- split
		- deplate
		- match

## 3 布尔值

下面值会被转换为 false

- undefined
- null
- 0
- -0
- NaN
- ""

## 4 null 和 undefined



## 5 全局对象

> 全局对象的属性是全局定义的符号，JS程序可以直接使用。当 JS 解释器启动时(任何 Web 游览器加载新页面的时候)，它将创建一个新的全局对象，并给它一组定义的初始属性：
>
> - 全局属性，比如 undefined、Infinity 和 NaN
> - 全局函数，比如 isNaN() parseInt() eval()
> - 构造函数，比如 Date()、RegExp() String()、Object() 和 Array() 
> - 全局对象，Math和 JSON
>
> 客户端 JavaScript 中，表示的游览器窗口中的所有 JS 代码中，Window 对象充当了全局对象。



## 6 包装对象

对象是一种复合值，当属性值是一个函数的时候，称为方法。存取字符串、数字或布尔值的属性时创建的临时对象称为包装对象，

## 7 不可变的原始值和可变的对象引用

```js
var s = "hello"
s.toUpperCase();//返回 HELLO，但并没有改变 s的值
```

- 两个对象包含同样的属性及相同的值，它们也是不相等的。对象是引用类型，当只有引用相同才会相等
- 想比较两个单独的对象和数组，则必须比较它们的属性或元素

## 8 类型转换

- 如果转换结果无意义的话将返回 NaN

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhgy1gkizzahbyzj30tw0owwpi.jpg)

### 1 转换与相等性

```js
// 下面结果都为 true
null==underfined 
"0"==0
0==false
"0"==false
```

### 2 显式类型转换

```js
// 使用 Boolean() Number()...函数
Number("3")
String(false)
Boolean([]) // ture 🤔？
Object(3)
```

- 除 null 或 undefined 之外的任何值都具有 toString()方法，这个方法执行结果通常和 String()方法的返回结果一致
- Number类定义的 toString()方法可以接受表示转换基数(radix)
	- `binary_string=n.toString(2);`// 转换为 "10001"
	- `octal_string="0"+n.toString(8)`;//转换为 021

### 3 对象转换为原始值

对象到布尔值的转换非常简单：所有对象都转换为 true。包装对象亦是如此: new Boolean（false） 是一个对象而不是原始值，它将转换为 true

```shell
({x:1,y:2}).toString() // => "[object Object]"
[1,2,3].toString() // "1,2,3"
```

## 9 变量声明

变量是使用关键字 var 来声明的，如下所示：

```js
var i;
var sum;
// or
var i,sum;
```

## 10 变量作用域



## 11 函数作用域和声明提前

