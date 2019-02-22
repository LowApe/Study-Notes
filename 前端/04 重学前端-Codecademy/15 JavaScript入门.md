# JavaScript入门

## 1. Console

对我们来说，将值打印到控制台非常有用，因此我们可以看到我们正在做的工作。

分号表示行的结尾或语句。虽然在JavaScript中你的代码通常会在没有分号的情况下按预期运行，但我们建议学习用分号结束每个语句的习惯，这样你就不会在需要的时候在少数几个实例中留下一个。
```js
console.log(5);
```

## 2. Comments 注释
1. 单行
2. 多行

```js
// Prints 5 to the console
console.log(5);

/*
This is all commented
console.log(10);
None of this is going to run!
console.log(99);
*/
```

## 3. Data Types 数据类型

数据类型是我们为编程中使用的不同类型的数据提供的分类。在JavaScript中，有七种基本数据类型：

1. 编号：任何号码，包括带有小数的数字：4，8，1516，23.42。
字符串：键盘上的任何字符组（字母，数字，空格，符号等），用单引号括起来：' ... '或双引号" ... "。虽然我们更喜欢单引号。有些人喜欢将字符串视为文本的一个奇特的词。
2. Boolean：此数据类型只有两个可能的值 - true或者false（没有引号）。将布尔值视为开关以及“是”或“否”问题的答案是有帮助的。
3. Null：此数据类型表示故意缺少值，并由关键字null（不带引号）表示。
4. 未定义：此数据类型由关键字undefined（不带引号）表示。它也表示没有值，尽管它有不同的用途null。
5. 符号：语言的新功能，符号是唯一标识符，在更复杂的编码中很有用。现在无需担心这些问题。
6. 对象：相关数据的集合。

这些类型中的前6个被认为是原始数据类型。它们是该语言中最基本的数据类型。对象更复杂，随着JavaScript的进展，您将学到更多关于它们的知识。起初，有七种类型可能看起来不那么多，但很快你会发现，一旦你开始利用每一种类型，世界就会开启。当您了解有关对象的更多信息时，您将能够创建复杂的数据集合。

但在我们这样做之前，让我们对字符串和数字感到满意！

## 4. Arithmetic Operators 算术运算符
编程时，基本算术通常会派上用场。

一个操作是执行在我们的代码任务的角色。`JavaScript`有几个内置的算术运算符，允许我们对数字执行数学计算。这些包括以下运算符及其相应的符号：

1. 加： +
2. 减去： -
3. 乘： *
4. 划分： /
5. 剩余： %

## 5. String Concatenation 字符串拼接

```js
console.log('hi' + 'ya'); // Prints 'hiya'
console.log('wo' + 'ah'); // Prints 'woah'
console.log('I love to ' + 'code.')
// Prints 'I love to code.'
```

```js
console.log('front ' + 'space');
// Prints 'front space'
console.log('back' + ' space');
// Prints 'back space'
console.log('no' + 'space');
// Prints 'nospace'
console.log('middle' + ' ' + 'space');
// Prints 'middle space'
```
## 6. Properties 属性

当您将新数据引入JavaScript程序时，浏览器会将其保存为数据类型的实例。每个字符串实例都有一个名为的属性length，用于存储该字符串中的字符数。您可以通过附加带有句点和属性名称的字符串来检索属性信息：
```js
console.log('Hello'.length); // Prints 5
```
这.是另一个运营商！我们称之为点运算符。

在上面的示例中，length从字符串的实例中检索保存到属性的值'Hello'。程序打印5到控制台，因为Hello它有五个字符。

## 7. Methods 方法
请记住，方法是我们可以执行的操作。JavaScript提供了许多字符串方法。

我们通过附加一个带有句点（点运算符）的实例，方法的名称以及开括号和右括号来调用或使用这些方法：即。'example string'.methodName()。

这种语法看起来有点熟悉吗？当我们使用时，console.log()我们在对象.log()上调用方法console。让我们看看console.log()和一些真正的字符串方法！

```js
console.log('hello'.toUpperCase()); // Prints 'HELLO'
console.log('Hey'.startsWith('H')); // Prints true

```

让我们看看上面的每一行：

在第一行，`.toUpperCase()`在字符串实例上调用该方法'hello'。结果将记录到控制台。此方法返回所有大写字母的字符串：'HELLO'。
在第二行，`.startsWith()`在字符串实例上调用该方法'Hey'。此方法还接受字符'H'作为括号之间的输入或参数。由于字符串'Hey'以字母开头'H'，因此该方法返回布尔值true。

## 8. Built-in Objects 内置对象
除此之外`console`，JavaScript还`内置`了其他对象。在线下，您将构建自己的对象，但是现在这些“内置”对象充满了有用的功能。

例如，如果您想执行比算术更复杂的数学运算，则JavaScript具有内置`Math`对象。

关于对象的好处是他们有方法！让我们`.random()`从内置Math对象中调用该方法：

```js
console.log(Math.random()); // Prints a random number between 0 and 1
```
在上面的示例中，我们.random()通过使用点运算符附加对象名称，方法名称以及左括号和右括号来调用方法。此方法返回0到1之间的随机数。

要生成0到50之间的随机数，我们可以将此结果乘以50，如下所示：

```js
Math.random() * 50;
```
面的示例可能会计算为小数。为了确保答案是一个整数，我们可以利用另一种有用的Math方法Math.floor()。

Math.floor()取十进制数，并向下舍入到最接近的整数。您可以使用Math.floor()这样舍入一个随机数：
```js
Math.floor(Math.random() * 50); //向下取整

Math.ceil(Math.random() * 50); //向上取整
```

# 总结

- 数据被打印或记录到控制台，一个显示消息的面板，用`console.log()`。
- 您可以//在/*和之间编写单行注释和多行注释*/。
- JavaScript中有7种基本数据类型：字符串，数字，布尔值，null，undefined，symbol和object。
- 数字是没有引号的任何数字： 23.8879
- 字符串是用单引号或双引号括起来的字符： 'Sample String'
- 内置的算术运算符包括`+`，`-`，`*`，`/`，和`%`。
- 对象（包括数据类型的实例）可以具有属性，存储的信息。属性.在对象名称后面用a表示，例如：`'Hello'.length`。
- 对象（包括数据类型的实例）可以具有执行动作的方法。通过使用句点，方法名称和括号附加对象或实例来调用方法。例如：`'hello'.toUpperCase()`。
- 我们可以使用`.dot`运算符访问属性和方法。
- 内置对象（包括Math）是`JavaScript`提供的方法和属性的集合。
