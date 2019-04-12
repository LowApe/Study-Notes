# JavaScript数组

> 参考数组的所有方法[MDN Array
](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)

```js
let newYearsResolutions = ['Keep a journal', 'Take a falconry class', 'Learn to juggle'];
```
## 1. .push（）方法

让我们了解一些内置的`JavaScript`方法，这些方法可以更轻松地处理数组。这些方法专门在数组上调用，以使常见任务（如添加和删除元素）更加直接。

一种方法，`.push()`允许我们将项`添加`到数组的末尾。以下是如何使用它的示例：

```js
const itemTracker = ['item 0', 'item 1', 'item 2'];

itemTracker.push('item 3', 'item 4');

console.log(itemTracker);
// Output: ['item 0', 'item 1', 'item 2', 'item 3', 'item 4'];
```
那么，`.push()`工作怎么样？

- 我们`push`使用点符号访问该方法，连接`push`到`itemTracker`句点。
- 然后我们将其称为函数。那是因为`.push()JavaScript`是一个允许我们在数组上使用的函数。
- .push()可以使用逗号分隔的单个参数或多个参数。在这种情况下，我们增加了两个元素：'item 3'和'item 4'来itemTracker。
- 请注意，`.push()`改变或变异，itemTracker。您可能还会看到`.push(`)称为破坏性数组方法，因为它会更改初始数组。


## 2. The .pop() Method
另一个数组方法，.pop()删除数组的最后一项。
```js
const newItemTracker = ['item 0', 'item 1', 'item 2'];

const removed = newItemTracker.pop();

console.log(newItemTracker);
// Output: [ 'item 0', 'item 1' ]
console.log(removed);
// Output: item 2
```

## 3. 更多方法
[更多方法:](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)
```js
.shift() //删除左边的
.unshift(xx) //向左添加内容
.splice(1,4) //通过索引删除某个元素 1-4
```

从一个索引位置删除多个元素
```js
var vegetables = ['Cabbage', 'Turnip', 'Radish', 'Carrot'];
console.log(vegetables);
// ["Cabbage", "Turnip", "Radish", "Carrot"]

var pos = 1, n = 2;

var removedItems = vegetables.splice(pos, n);
// this is how to remove items, n defines the number of items to be removed,
// from that position(pos) onward to the end of array.

console.log(vegetables);
// ["Cabbage", "Carrot"] (the original array is changed)

console.log(removedItems);
// ["Turnip", "Radish"]
```

## 4. Nested Arrays 嵌套数组
之前我们提到过数组可以存储其他数组。当数组包含另一个数组时，它被称为嵌套数组。检查以下示例：

```js
const nestedArr = [[1], [2, 3]];
console.log(nestedArr[1]); // Output: [2, 3]
console.log(nestedArr[1][0]); // Output: 2  

```

# 总结

- 数组是用`JavaScript`存储数据的列表。
- 使用括号创建数组`[]`。
- 数组内的每个项目都位于编号的位置或索引处`0`。
- 我们可以使用索引访问数组中的一个项目，语法如下：`myArray[0]`。
- 我们还可以使用索引更改数组中的项，语法如下`myArray[0] = 'new string'`;
- 数组有一个`length`属性，允许您查看数组中有多少项。
- 数组有自己的方法，包括`.push()`和`.pop()`分别添加和删除数组中的项目。
- 阵列有一个执行不同的任务，比如很多方法`.slice()`和`.shift()`，你可以在找到[Mozilla开发者网络](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)。
- 一些内置方法正在变异，这意味着该方法将改变数组，而其他方法则不会发生变异。您始终可以查看文档。
- 包含数组的变量可以用`let`或声明`const`。即使在声明时`const`，数组仍然是可变的。但是，声明的变量`const`无法重新分配。
- 在函数内部变异的数组将在函数外部保持该变化。
- 数组可以嵌套在其他数组中。
- 使用括号表示法访问嵌套数组链索引中的元素。
