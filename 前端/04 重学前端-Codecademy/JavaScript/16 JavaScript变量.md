# JavaScript 变量

## 1. Create a Variable: var
2015年，ES6版本的JavaScript中引入了许多更改。最大的变化之一是两个新关键字，`let`以及`const`创建或声明变量。在ES6之前，程序员只能使用var关键字来声明变量。

```js
var myName = 'Arya';
console.log(myName);
// Output: Arya
```
命名变量有一些通用规则：

- 变量名称不能以`数字`开头。
- 变量名称是`区分大小`写的，因此`myName`和`myname`将不同的变量。使用不同的情况创建两个具有相同名称的变量是不好的做法。
- 变量名称不能与关键字相同。有关关键字的完整列表，请查看[关键字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Keywords)

## 2. Create a Variable: let 创建变量：let
正如上一个练习中所提到的，`let`关键字是在ES6中引入的。在`let`该变量可以被重新分配一个不同的值的关键字信号。看看这个例子：
```js
let meal = 'Enchiladas';
console.log(meal); // Output: Enchiladas
meal = 'Burrito';
console.log(meal); // Output: Burrito
```
在使用`let`（甚至var）时我们应该注意的另一个概念是我们可以声明一个变量而不为变量赋值。在这种情况下，变量将自动初始化为以下值`undefined`：

```js
let price;
console.log(price); // Output: undefined
price = 350;
console.log(price); // Output: 350
```
请注意上面的例子：

如果我们不为使用`le`t关键字声明的变量赋值，则它自动具有值`undefined`。
我们可以重新分配变量的值。

##　3. Create a Variable: const 创建一个变量：const

该`const`关键字也在ES6中引入，并且是单词常量的缩写。就像使用`var`，`let`您可以将任何值存储在`const`变量中。声明`const`变量并为其赋值的方式遵循与`let`和相同的结构`var`。看一下下面的例子：

```js
const myName = 'Gilberto';
console.log(myName); // Output: Gilberto
```

但是，`const`无法重新分配变量，因为它是常量。如果你试图重新分配一个`const`变量，你会得到一个`TypeError`。

声明时必须为常量变量赋值。如果你试图声明一个`const`没有值的变量，你会得到一个SyntaxError。

如果您正在尝试决定使用哪个关键字，`let`或者`const`考虑以后是否需要重新分配变量。如果确实需要重新分配变量`let`，请使用`const`。

## 4. Mathematical Assignment Operators
让我们考虑如何使用变量和数学运算符来计算新值并将它们分配给变量。看看下面的例子：

```js
let w = 4;
w = w + 1;

console.log(w); // Output: 5
```
在上面的示例中，我们`w`使用4分配给它的编号创建了变量。以下行将from w = w + 1的值增加到5。

w在对其执行某些数学运算之后我们可以重新分配的另一种方法是使用内置的数学赋值运算符。我们可以重新编写上面的代码：

```js
let w = 4;
w += 1;

console.log(w); // Output: 5
```

我们还可以使用其他数学赋值运算符：`-=`，`*=`，和`/=`其工作以类似的方式。

```js
let x = 20;
x -= 5; // Can be written as x = x - 5
console.log(x); // Output: 15

let y = 50;
y *= 2; // Can be written as y = y * 2
console.log(y); // Output: 100

let z = 8;
z /= 2; // Can be written as z = z / 2
console.log(z); // Output: 4
```

## 5. The Increment and Decrement Operator 增量和减量运算符
 `++`  `--` 用法
```js

let a = 10;
a++;
console.log(a); // Output: 11

let b = 20;
b--;
console.log(b); // Output: 19
```

## 6. String Concatenation with Variables
在之前的练习中，我们为变量分配了字符串。现在，我们来看看如何连接或连接变量中的字符串。

`+`即使这些值存储在变量中，运算符也可用于组合两个字符串值：
```js
let myPet = 'armadillo';
console.log('I own a pet ' + myPet + '.');
// Output: 'I own a pet armadillo.'
```

## 7. String Interpolation 字符串插值
在ES6版本的JavaScript中，我们可以使用模板文字将变量插入或插入到字符串中。请查看以下示例，其中使用模板文字将字符串记录在一起：
```js
const myPet = 'armadillo';
console.log(`I own a pet ${myPet}.`);
// Output: I own a pet armadillo.
```

请注意：

模板文字由反引号包裹```（此键通常位于键盘顶部，键的左侧1）。
在模板文字内，你会看到一个占位符`${myPet}`。值的值`myPet`插入到模板文字中。
当我们插值时`I own a pet ${myPet}.`，我们打印的输出是字符串：`I own a pet armadillo.`

使用模板文字的最大好处之一是代码的可读性。使用模板文字，您可以更轻松地告诉新字符串是什么。您也不必担心转义双引号或单引号。

----

## 8. typeof operator typeof运算符

在编写代码时，跟踪程序中变量的数据类型会很有用。如果需要检查变量值的数据类型，可以使用`typeof`运算符。

该`typeof`操作员检查值在其右侧和回报，或者传递回，该数据类型的字符串。

```js
onst unknown1 = 'foo';
console.log(typeof unknown1); // Output: string

const unknown2 = 10;
console.log(typeof unknown2); // Output: number

const unknown3 = true;
console.log(typeof unknown3); // Output: boolean
```

# 总结

- 变量在程序中保存可重用数据并将其与名称相关联。
- 变量存储在内存中。
- 该`var`关键字用于`JS`的`ES6`之前的版本。
- `let`是可以重新分配变量时声明变量`const`的首选方法，并且是声明具有常量值的变量的首选方法。
- 尚未初始化的变量存储原始数据类型`undefined`。
- 数学赋值运算符可以轻松计算新值并将其分配给同一变量。
- 该`+`运算符用于连接字符串包括变量举行的字符串值
- 在`ES6`中，模板文字使用反引号`并将`${}`值插入到字符串中。
- 的`typeof`关键字返回一值的数据类型（字符串）。


## JavaScript版本：ES6和之前版本

您可能已经看过“ES6”或“Javascript ES6”一词，并想知道它的实际含义。不用多说了，因为我们将深入研究ES6是什么以及它与Javascript的关系！

首先，让我们介绍一些历史。JavaScript于1995年由Netscape Communications公司引入，作为网页设计师和程序员与网页交互的脚本语言。第二年，Netscape向一家名为Ecma International的标准开发组织提交了JavaScript，以创建脚本语言（一种编程语言）的标准。1997年，Ecma International发布了ECMA-262，它为第一版称为ECMAScript的脚本语言设定了标准，缩写为ES。

这些新的ECMAScript标准为JavaScript功能的体系结构提供了规则。随着新的编程范例的出现以及开发人员寻求新功能，新版本的ECMAScript为新旧JavaScript版本之间的一致性提供了基础。

要完全区分JavaScript和ECMAScript之间的区别：
- 如果要创建应用程序或程序，可以使用JavaScript
- 如果要创建新的脚本语言，可以遵循ECMAScript中的指导原则。

因此，当您看到ES6或JavaScript ES6时，这意味着该版本的JavaScript遵循第六版ECMAScript中的规范！您可能还会看到ES2015而不是ES6，但这两个术语都指的是2015年发布的第6版ECMAScript。请查看下面的时间表，了解JavaScript多年来的演变情况：

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0iwlni69nj30xq0q5afn.jpg)

时间表显示JS版本从开始到2018年的演变
现在，您可能会问，当ES7和ES8等更新的更新时，2015年的更新仍然与今天有关？

好吧，尽管发布了更新版本，ES6实际上是自1997年第一版发布以来对ECMAScript的最大更新！一些开发人员甚至将ES6称为“现代JavaScript”，因为所有主要的增加。添加了许多强大的功能来帮助JavaScript开发人员，包括：

新的关键字喜欢let和const声明变量，
使用Arrow函数的新函数语法，
创建类，
具有默认值的参数，
承诺异步操作，
还有很多！
最新的浏览器现在支持大多数ES6功能，允许开发人员利用这些新增功能。ES6最终允许程序员节省时间并编写更简洁的代码。以函数表达式的ES6前语法为例：

```js

var  greeting  =  function()
{
    console。log('Hello World！');
} ;

```

使用ES6箭头函数，我们可以将上面的表达式转换为：

```js
const  greeting  = ()=> console.log（'Hello World'）;   
```

但是，箭头函数不仅仅是语法重写。与其他ES6功能一样，还有其他潜在的好处和权衡需要考虑。尽管如此，ES6在开发社区中得到了广泛采用。诸如新的ES6语法之类的好处使得使用流行的编程范例（面向对象编程（OOP））变得更加容易。这种变化允许习惯于OOP的其他语言的开发人员现在可以转换为学习和使用JavaScript。ES6流行的另一个原因与在React等流行框架中使用ES6有关。因此，如果您想学习最新的工具和框架，您将不得不在此过程中学习ES6。

话虽如此，我们不应该忽视遗留代码，即旧版本的JavaScript。实际上，仍有许多项目是使用遗留代码构建和维护的！如果您希望能够自由地处理任何类型的JavaScript项目，那么您应该熟悉pre-ES6和ES6 JavaScript语法。但不要担心，我们在JavaScript课程中涵盖了ES6和ES6之前的版本。看看它成为JavaScript基础知识的摇滚明星，并学习基本的编程技巧！
