# JavaScript function
首次学习如何计算矩形区域时，有一系列步骤来计算正确的答案：

- 测量矩形的宽度。
- 测量矩形的高度。
- 乘以矩形的宽度和高度。
- 通过练习，您可以计算矩形的面积，而无需每次都指示这三个步骤。

```js
const width = 10;
const height = 6;
const area =  width * height;
console.log(area); // Output: 60

// Area of the first rectangle
const width1 = 10;
const height1 = 6;
const area1 =  width1 * height1;

// Area of the second rectangle
const width2 = 4;
const height2 = 9;
const area2 =  width2 * height2;

// Area of the third rectangle
const width3 = 10;
const height3 = 10;
const area3 =  width3 * height3;
```

## 1. 函数声明

在JavaScript中，有许多方法可以创建函数。创建函数的一种方法是使用函数声明。就像变量声明如何将值绑定到变量名称一样，函数声明将函数绑定到`名称`或`标识符`。看看下面的函数声明的解剖：

该图显示了函数声明的语法
函数声明包括：

- 该function关键字。
- 函数的名称或其标识符，后跟括号。
- 函数体，或执行特定任务所需的语句块，包含在函数的花括号中{ }。
函数声明是绑定到标识符或名称的函数。在下一个练习中，我们将讨论如何在函数体内运行代码。

我们还应该了解JavaScript 中的提升功能，它允许在定义函数声明之前访问它们。

看一下吊装的例子：
```js
console.log(greetWorld()); // Output: Hello, World!

function greetWorld() {
  console.log('Hello, World!');
}
```

## 2. Calling a Function 调用函数

```js
function greetWorld() {
  console.log('Hello, World!');
}
greetWorld();
```

## 3. Parameters and Arguments 参数

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0jtgw1yggj30an04j74h.jpg)
函数头部是实际参数,函数内部是具体形参

## 4. Default Parameters 默认参数
ES6中添加的功能之一是能够使用默认参数。默认参数允许参数具有预定值，以防没有参数传递给函数或者参数是undefined在调用时。

看一下下面使用默认参数的代码片段：
```js
function greeting (name = 'stranger') {
  console.log(`Hello, ${name}!`)
}

greeting('Nick') // Output: Hello, Nick!
greeting() // Output: Hello, stranger!
```
- 在上面的示例中，我们使用`=`运算符为参数`name`指定默认值 `'stranger'`。如果我们想要包含非个性化的默认问候语，这非常有用！

- 当代码调用`greeting('Nick')`参数的值传入和时`'Nick'`，将覆盖默认参数`'stranger'`以记录`'Hello, Nick!'`到控制台。

- 如果没有传入参数`greeting()`，`'stranger'`则使用默认值，并将`'Hello, stranger!'`其记录到控制台。

## 5. Return
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0jugprhk9j30am03z74e.jpg)

```js
function rectangleArea(width, height) {
  if (width < 0 || height < 0) {
    return 'You need positive integers to calculate area!';
  }
  return width * height;
}
```

## 6. Helper Functions

我们也可以在另一个函数中使用函数的返回值。在另一个函数中调用这些函数通常称为辅助函数。由于每个函数都执行特定任务，因此在必要时使代码更易于阅读和调试。


```js
function multiplyByNineFifths(number) {
  return number * (9/5);
};

function getFahrenheit(celsius) {
  return multiplyByNineFifths(celsius) + 32;
};

getFahrenheit(15); // Returns 59
```
## 7. Function Expressions功能表达
定义函数的另一种方法是使用函数表达式。要在表达式中定义函数，我们可以使用function关键字。在函数表达式中，通常省略函数名称。没有名称的函数称为匿名函数。函数表达式通常存储在变量中以便引用它。

考虑以下函数表达式：

定义函数表达式
声明一个函数表达式：

声明一个变量，使变量的名称成为函数的名称或标识符。自ES6发布以来，通常的做法是使用const关键字来声明变量。

将该变量的值指定为使用function关键字后跟一组带有可能参数的括号创建的`匿名函数`。然后是一组包含函数体的花括号。

要调用函数表达式，请写入存储函数的变量的名称，然后用括号括起要传递给函数的任何参数。

```js
const plantNeedsWater = function(day){

}
```

## 8. Arrow Functions 箭头函数

`ES6`引入了箭头函数语法，这是一种使用特殊“()箭头” `() =>`表示法编写函数的简短方法。

`function`每次需要创建函数时，箭头函数都不需要键入关键字。相反，您首先在其中包含参数`( )`，然后添加`=>`指向包围的函数体的箭头，{ }如下所示：
```js
const rectangleArea = (width, height) => {
  let area = width * height;
  return area;
}
```

## 9. Concise Body Arrow Functions 简洁的身体箭头功能
`JavaScript`还提供了几种重构箭头函数语法的方法。最简洁的功能形式称为简洁的身体。我们将在下面探讨以下几种技术：

1. 仅采用单个参数的函数不需要将该参数括在括号中。但是，如果函数采用零个或多个参数，则需要括号。

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0jwjawbs0j30cz05laae.jpg)
2. 由单行块组成的函数体不需要花括号。如果没有花括号，则会自动返回该行评估的内容。块的内容应紧跟箭头，`=>`并且`return`可以删除关键字。这称为`隐式返回`。

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0jwlpmxbhj30bz05cjro.jpg)

```js
const squareNum = (num) => {
  return num * num;
};
```

```js
const squareNum = num => num * num;
```
- 周围的括号`num `已被删除，因为它有一个参数。
- 花括号`{ }`已被删除，因为该函数由单行块组成。
- 该`return`关键字已被删除，因为该函数由单行块组成。

# 总结


- ES6引入了通过默认参数处理任意参数的新方法，这允许我们在没有参数传递给函数的情况下为参数分配默认值。
- 要从函数返回值，我们使用`return`语句。
- 使用函数表达式定义函数：

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0jwpp6miyj30cd04at8w.jpg)

- 使用箭头函数表示法定义函数：

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0jwqiooc4j30b80490su.jpg)

- 使用简洁的箭头符号可以使函数定义简洁：

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0jwqtal6ij30cw05iwes.jpg)
