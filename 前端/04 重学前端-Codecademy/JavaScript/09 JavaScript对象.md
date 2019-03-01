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
