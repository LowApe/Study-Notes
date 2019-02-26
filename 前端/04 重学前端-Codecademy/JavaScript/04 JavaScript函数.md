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
