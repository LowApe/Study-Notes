# JavaScript迭代器
想象一下，你有一个购物清单，你想知道清单上的每个项目是什么。您必须扫描每一行并检查该项目。这个常见任务类似于我们想要迭代或循环遍历数组时要做的事情。我们可以使用的一个工具是for循环。但是，我们还可以访问内置的数组方法，使循环更容易。

帮助我们迭代的内置JavaScript数组方法称为迭代方法，有时称为迭代器。迭代器是在数组上调用的方法，用于操作元素和返回值。

在本课程中，您将学习这些方法的语法，它们的返回值，如何使用文档来理解它们，以及如何为给定任务选择正确的迭代器方法。

- .forEach()
- .map()
- .filter()

## 1. The .forEach() Method
我们要学习的第一个迭代方法是`.forEach()`。`.forEach()`将为数组的每个元素执行相同的代码。
概述数组迭代器部分的图表，包括数组标识符，迭代器部分和回调函数

上面的代码会将格式良好的杂货列表记录到控制台。让我们来探索一下调用的语法`.forEach()`。

- `groceries.forEach()`forEach在groceries数组上调用方法。
- `.forEach()`采用回调函数的参数。请记住，回调函数是作为参数传递给另一个函数的函数。
- `.forEach()`循环遍历数组并为每个元素执行回调函数。在每次执行期间，当前元素作为参数传递给回调函数。
- 返回值`.forEach()`始终为`undefined`。
传递回调的另一种方法`.forEach()`是使用箭头函数语法。

```js
groceries.forEach(groceryItem => console.log(groceryItem));

function printGrocery(element){
  console.log(element);
}

groceries.forEach(printGrocery);
```

## 2. The .map() Method
我们要介绍的第二个迭代器是`.map()`。在`.map()`数组上调用时，它接受一个回调函数的参数并返回一个`新数组`！看一下调用的例子.map()：
```js

const numbers = [1, 2, 3, 4, 5];
const bigNumbers = numbers.map(number => {
  return number * 10;
});

```

`.map()`以类似的方式工作`.forEach()`- 主要区别是`.map()`返回一个新数组。

在上面的例子中：

- `numbers` 是一组数字。
- `bigNumbers`将存储调用的返回值`.map()`上`numbers`。
- `numbers.map`将遍历`numbers`数组中的每个元素并将该元素传递给回调函数。
- `return number * 10`是我们希望对数组中的每个元素执行的代码。这会将numbers数组中的每个值（乘以）保存10到新数组中。

## 3. The .filter() Method
`.filter（）`方法
另一个有用的迭代器方法是`.filter()`。比如`.map()`，`.filter()`返回一个新数组。但是，`.filter()`在从原始数组中`过滤掉某些元素`后返回一个元素`数组`。该`.filter()`方法的回调函数应该返回true或false取决于传递给它的元素。导致回调函数返回的元素`true`将添加到新数组中。看一下下面的例子：

```js
const words = ['chair', 'music', 'pillow', 'brick', 'pen', 'door'];
const shortWords = words.filter(word => {
  return word.length < 6;
});
```

```js
//运行结果
console.log(words); // Output: ['chair', 'music', 'pillow', 'brick', 'pen', 'door'];
console.log(shortWords); // Output: ['chair', 'music', 'brick', 'pen', 'door']
```

- `words `是一个包含字符串元素的数组。
- `const shortWords` 声明一个新变量，它将存储从调用返回的数组`.filter()`。
- 回调函数是一个箭头函数，有一个参数，word。words数组中的每个元素都将作为参数传递给此函数。
- word.length < 6;是回调函数中的条件。word来自words数组少于6字符的任何数组都将添加到shortWords数组中。

## 4. The .findIndex() 方法

我们有时想在数组中找到元素的位置。这就是`.findIndex()`方法的用武之地！调用.findIndex()数组将返回true在回调函数中求值的第一个元素的索引。

```js
const jumbledNums = [123, 25, 78, 5, 9];

const lessThanTen = jumbledNums.findIndex(num => {
  return num < 10;
});
```
```js
console.log(lessThanTen); // Output: 3
```

```js
console.log(jumbledNums[3]); // Output: 5
```
- jumbledNums 是一个包含数字元素的数组。
- const lessThanTen =声明一个新变量，用于存储从调用返回的索引号.findIndex()。
- 回调函数是一个箭头函数，有一个参数，num。jumbledNums数组中的每个元素都将作为参数传递给此函数。
- num < 10;是检查元素的条件。.findIndex()将返回true为该条件求值的第一个元素的索引。
让我们来看看lessThanTen评估的内容：


```js
const greaterThan1000 = jumbledNums.findIndex(num => {
  return num > 1000;
});

console.log(greaterThan1000); // Output: -1
```
如果数组中没有单个元素满足回调中的条件，则.findIndex()返回-1。
## 5. The .reduce() Method
另一种广泛使用的迭代方法是`.reduce()`。`.reduce()`迭代遍历数组的元素后，该方法返回单个值，从而减少数组。看看下面的例子：

```js
const numbers = [1, 2, 4, 10];

const summedNums = numbers.reduce((accumulator, currentValue) => {
  return accumulator + currentValue
})

console.log(summedNums) // Output: 17
```

以下是迭代数组的值accumulator和：currentValuenumbers
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0mcxvit7fj30g204jmx4.jpg)

- `numbers` 是一个包含数字的数组。
- `summedNums`是存储在调用的返回值的变量`.reduce()`上`numbers`。
- `numbers.reduce()`调用数组`.reduce()`上的方法`numbers`并接受回调函数作为参数。
- 回调函数有两个参数，`accumulator`和`currentValue`。值`accumulator`以数组中第一个元素的值`currentValue`开始，以第二个元素开始。要查看accumulator和currentValue更改的值，请查看上面的图表。
- 当`.reduce()`遍历数组时，回调函数的返回值变为accumulator下一次迭代`currentValue`的值，在循环过程中采用当前元素的值。

----

```js
const numbers = [1, 2, 4, 10];

const summedNums = numbers.reduce((accumulator, currentValue) => {
  return accumulator + currentValue
}, 100)  // <- Second argument for .reduce()

console.log(summedNums); // Output: 117
```
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0mdkvmmaxj30gj05ht8q.jpg)
解释 100 为初始值

## 6. Iterator Documentation 迭代器文档

还有许多其他内置数组方法，其完整列表位于 MDN的Array迭代方法页面上。

每种方法的文档包含几个部分：

1. 一个简短的定义。
2. 具有使用该方法的正确语法的块。
3. 方法接受或要求的参数列表。
4. 函数的返回值。
5. 扩展说明。
6. 该方法的使用示例。
7. 其他附加信息。

# 总结



- `.forEach()`用于在数组中的每个元素上执行相同的代码，但不更改数组并返回`undefined`。
- `.map() `在数组中的每个元素上执行相同的代码，并返回带有更新元素的新数组。
- `.filter()` 检查数组中的每个元素以查看它是否满足特定条件并返回一个新数组，其中包含返回条件的truthy元素。
- `.findIndex()`返回满足回调函数条件的数组的第一个元素的索引。它返回-1如果没有一个阵列中的元件的满足条件。
- `.reduce() `遍历数组并获取元素的值并返回单个值。
- 所有迭代器方法都采用可以预定义的回调函数，或函数表达式或箭头函数。
您可以访问Mozilla Developer Network以了解有关迭代器方法（以及JavaScript的所有其他部分！）的更多信息。
