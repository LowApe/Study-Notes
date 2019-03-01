# JavaScript对象
你可能已经比你想象的更熟悉对象，因为JavaScript喜欢对象！语言的许多组成部分实际上都是对象，甚至不像字符串或数字的部分在某些情况下仍然可以像对象一样。

JavaScript中只有七种基本数据类型，其中六种是原始数据类型：`字符串`，`数字`，`布尔值`，`空值`，`未定义`和`符号`。使用第七种类型的对象，我们打开代码以获得更复杂的可能性。我们可以使用JavaScript对象来模拟真实世界的事物，比如篮球，或者我们可以使用对象来构建使Web成为可能的数据结构。

JavaScript对象的核心是存储相关数据和功能的容器，但这种看似简单的任务在实践中非常强大。你一直在使用物体的力量，但是现在是时候理解物体的力学并开始制造自己的物体了！

## 1. Creating Object Literals 创建对象文字
可以像任何 JavaScript 类型一样将对象分配给变量。我们使用花括号`{}`来指定对象文字：

```js
let spaceship = {}; // spaceship is an empty object
```

我们用无序数据填充对象。该数据被组织成键值对。键就像一个变量名，指向内存中保存值的位置。
![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0mhtls0c0j309n04rmx6.jpg)

我们通过写入密钥的名称或标识符，然后是冒号然后是值来创建键值对。我们用逗号（,）分隔对象文字中的每个键值对。键是字符串，但是当我们有一个没有任何特殊字符的键时，JavaScript允许我们省略引号：

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0mhv2x18jj30cn06hjri.jpg)

该`spaceship`对象有两个属性`Fuel Type`和`color`。'Fuel Type'有引号，因为它包含空格字符。
```js
let spaceship = {
    'name':'Tome',
    'color':'red'
};
```
## 2. Accessing Properties 访问属性
我们可以通过两种方式访问​​对象的属性。让我们探索第一种方式 - 点符号`，`.。您已使用点表示法来访问内置对象和数据实例的属性和方法：

```js
'hello'.length; // Returns 5

```

```js
let spaceship = {
  homePlanet: 'Earth',
  color: 'silver'
};
spaceship.homePlanet; // Returns 'Earth',
spaceship.color; // Returns 'silver',
```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0mi06sikzj308o04l747.jpg)

## 3. Bracket Notation 括号表示法
访问键值的第二种方法是使用括号表示法`[ ]`<br>
索引数组时使用括号表示法：
```js
['A', 'B', 'C'][0]; // Returns 'A'
```
要使用括号表示法访问对象的属性，我们将属性名称（键）作为字符串传递

当访问包含数字，空格或特殊字符的键时，我们必须*使用括号表示法。在这些情况下没有括号表示法，我们的代码会抛出错误。

```js
let spaceship = {
  'Fuel Type': 'Turbo Fuel',
  'Active Duty': true,
  homePlanet: 'Earth',
  numCrew: 5
};
spaceship['Active Duty'];   // Returns true
spaceship['Fuel Type'];   // Returns  'Turbo Fuel'
spaceship['numCrew'];   // Returns 5
spaceship['!!!!!!!!!!!!!!!'];   // Returns undefined
```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0ndz9vo8yj309704ca9y.jpg)

```js
let returnAnyProp = (objectName, propName) => objectName[propName];

returnAnyProp(spaceship, 'homePlanet'); // Returns 'Earth'
```

## 4. Property Assignment 属性分配

一旦我们定义了一个对象，我们就不会遇到我们编写的所有属性。对象是可变的意味着我们可以在创建它们之后更新它们！

我们可以使用
- `点表示法.`
- `括号表示法[]`
- `赋值运算符=`
<br>
来向对象添加新的键值对或更改现有属性。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0nebncu1vj309n038t8o.jpg)

可能会发生以下两种情况之一：

- 如果对象上已存在该属性，则之前保存的任何值都将替换为新分配的值。
- 如果没有具有该名称的属性，则会将新属性添加到该对象。

重要的是要知道虽然我们不能重新分配声明的对象`const`，但我们仍然可以改变它，这意味着我们可以添加新属性并更改那里的属性。

```js
const spaceship = {type: 'shuttle'};
spaceship = {type: 'alien'}; // TypeError: Assignment to constant variable.
spaceship.type = 'alien'; // Changes the value of the type property
spaceship.speed = 'Mach 5'; // Creates a new key of 'speed' with a value of 'Mach 5'
```

```js
const spaceship = {
  'Fuel Type': 'Turbo Fuel',
  homePlanet: 'Earth',
  mission: 'Explore the universe'
};

delete spaceship.mission;  // Removes the mission property
```

## 5. Methods 方法
当存储在对象上的数据是`函数`时，我们称之为`方法`。属性是对象具有的属性，而方法是对象所具有的属性。

对象方法似乎很熟悉吗？那是因为你一直在使用它们！例如，`console`是一个全局`javascript`对象，并且`.log()`是该对象的方法。`Math`也是一个全局`javascript`对象，并且`.floor()`是一个方法。

我们可以通过创建普通的，以逗号分隔的键值对来在对象文字中包含方法。键用作方法的名称，而值是匿名函数表达式。

```js
const alienShip = {
  invade: function () {
    console.log('Hello! We have come to dominate your planet. Instead of Earth, it shall be called New Xaculon.')
  }
};
```
使用ES6中引入的新方法语法，我们可以省略冒号和function关键字。

```js
const alienShip = {
  invade () {
    console.log('Hello! We have come to dominate your planet. Instead of Earth, it shall be called New Xaculon.')
  }
};
```

通过使用点运算符后跟方法名称和括号来附加对象的名称来调用对象方法：

```js
alienShip.invade(); // Prints 'Hello! We have come to dominate your planet. Instead of Earth, it shall be called New Xaculon.'
```

## 6. Nested Objects 嵌套对象
在应用程序代码中，对象通常是嵌套的 - 对象可能有另一个对象作为属性，而属性又可以具有更多对象的数组！

在我们的 `spaceship`对象中，我们想要一个`crew`对象。这将包含所有在船上做重要工作的船员。这些`crew`成员中的每一个都是对象本身。他们有样特性`name`，并`degree`和他们每个人都有根据自己的角色独特的方法。我们还可以嵌套其他对象，`spaceship`例如`telescope`关于父nanoelectronics对象内的太空船计算机的嵌套细节。

```js
const spaceship = {
     telescope: {
        yearBuilt: 2018,
        model: '91031-XLT',
        focalLength: 2032
     },
    crew: {
        captain: {
            name: 'Sandra',
            degree: 'Computer Engineering',
            encourageTeam() { console.log('We got this!') }
         }
    },
    engine: {
        model: 'Nimbus2000'
     },
     nanoelectronics: {
         computer: {
            terabytes: 100,
            monitors: 'HD'
         },
        'back-up': {
           battery: 'Lithium',
           terabytes: 50
         }
    }
};
```
我们可以链接运算符来访问嵌套属性。我们必须注意在每一层中使用哪种运算符。假装您是计算机并从左到右评估每个表达式以使每个操作开始感觉更易于管理可能会有所帮助。

```js
spaceship.nanoelectronics['back-up'].battery; // Returns 'Lithium'
```

在上面的代码中：

- 首先，计算机进行评估`spaceship.nanoelectronics`，从而生成包含`back-up`和`computer`对象的对象。
- 我们`back-up`通过追加来访问该 对象`['back-up']`。
- 该 `back-up`对象有一个`battery`属性，访问该属性`.battery`返回存储在那里的值：`'Lithium'`

## 7. Pass By Reference 通过参数传递
对象通过引用传递。这意味着当我们将分配给对象的变量作为`参数`传递给函数时，计算机会将参数名称解​​释为指向保存该对象的内存中的空间。因此，更改对象属性的函数实际上会永久地改变对象（即使将对象分配给 const 变量）。

```js
const spaceship = {
  homePlanet : 'Earth',
  color : 'silver'
};

let paintIt = obj => {
  obj.color = 'glorious gold'
};

paintIt(spaceship);

spaceship.color // Returns 'glorious gold'
```
我们的功能`paintIt()`永久地改变了我们`spaceship`对象的颜色。但是，重新分配 `spaceship`变量不会以相同的方式工作：

```js
let spaceship = {
  homePlanet : 'Earth',
  color : 'red'
};
let tryReassignment = obj => {
  obj = {
    identified : false,
    'transport type' : 'flying'
  }
  console.log(obj) // Prints {'identified': false, 'transport type': 'flying'}

};
tryReassignment(spaceship) // The attempt at reassignment does not work.
spaceship // Still returns {homePlanet : 'Earth', color : 'red'};

spaceship = {
  identified : false,
  'transport type': 'flying'
}; // Regular reassignment still works.
```

让我们看一下代码示例中发生的事情：

- 我们用这个`spaceship`对象声明了let。这使我们重新分配给一个新的对象`identified`和`'transport type'`性质，没有任何问题。
- 当我们使用旨在重新分配传递给它的对象的函数尝试相同的事情时，重新分配没有坚持（即使调用`console.log()`对象产生了预期的结果）。
- 当我们通过`spaceship`到该功能，`obj`成为对象的存储位置的参考`spaceship`对象，但不给`spaceship`变量。这是因为函数的`obj`参数`tryReassignment()`本身就是一个变量。身体`tryReassignment()`根本不知道`spaceship`变量！
- 当我们在主体中进行重新分配时 `tryReassignment()`，`obj`变量开始引用对象的内存位置{'identified' : false, 'transport type' : 'flying'}，而`spaceship`变量与其早期值完全不变。

## 8. Looping Through Objects循环通过对象
循环是编程工具，它们重复一段代码直到满足条件。我们学会了如何使用数字索引迭代数组，但是对象中的键值对没有排序！ `JavaScript`为我们提供了使用`for...in`语法迭代对象的替代解决方案。
`for...in` 将为对象中的每个属性执行给定的代码块。
```js
let spaceship = {
    crew: {
    captain: {
        name: 'Lily',
        degree: 'Computer Engineering',
        cheerTeam() { console.log('You got this!') }
        },
    'chief officer': {
        name: 'Dan',
        degree: 'Aerospace Engineering',
        agree() { console.log('I agree, captain!') }
        },
    medic: {
        name: 'Clementine',
        degree: 'Physics',
        announce() { console.log(`Jets on!`) } },
    translator: {
        name: 'Shauna',
        degree: 'Conservation Science',
        powerFuel() { console.log('The tank is full!') }
        }
    }
};
// for...in
for (let crewMember in spaceship.crew) {
  console.log(`${crewMember}: ${spaceship.crew[crewMember].name}`)
};
```

我们`for...in`将遍历`spaceship.crew`对象的每个元素。在每次迭代中，变量`crewMember`都设置为其中一个`spaceship.crew`键，使我们能够记录一个机组成员的角色列表`name`。

# 总结
您正在了解`JavaScript`中对象的机制。通过构建自己的对象，您将更好地理解`JavaScript`内置对象的工作原理。您还可以开始想象将代码组织到对象中并在代码中对现实世界进行建模。

让我们回顾一下我们在本课中学到的内容：

- 对象存储`键值对`的集合。
- 每个键值对都是一个属性 - 当属性是一个函数时，它被称为方法。
- 对象文字由以花括号括起的`逗号`分隔的`键值对`组成。
- 您可以使用`点表示法`或`括号表示法访问`，添加或编辑对象中的属性。
- 我们可以使用键值语法将匿名函数表达式作为值或使用新的`ES6`方法语法向我们的对象文字添加方法。
- 我们可以通过链接运算符来导航复杂的嵌套对象。
- 对象是可变的 - 即使它们被声明，我们也可以改变它们的属性`const`。
- 对象通过引用传递 - 当我们对传递给函数的对象进行更改时，这些更改是永久性的。
- 我们可以使用`For...in`语法迭代对象。
