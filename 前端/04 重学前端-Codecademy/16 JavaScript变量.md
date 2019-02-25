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
