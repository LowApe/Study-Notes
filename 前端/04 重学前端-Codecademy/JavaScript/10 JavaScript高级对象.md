# JavaScript 高级对象
## 1. 高级对象简介
请记住，`JavaScript`中的对象是`存储数据`和`功能`的容器。在本课中，我们将基于创建对象的基础知识并探索一些高级概念。

因此，如果没有异议，让我们更多地了解对象！

在本课中，我们将介绍以下主题：

- 如何使用`this`关键字。
- 用`JavaScript`方法传达隐私。
- 定义对象中的`getter`和`setter`。
- 创建`工厂`功能。
- 使用`解构`技术。

## 2. The This keyWord This 关键字

```js
const goat = {
  dietType: 'herbivore',
  makeSound() {
    console.log('baaa');
  }
};
```

```js
goat.makeSound(); // Prints baaa
```
在我们的`goat`对象中，我们有一个`.makeSound()`方法。我们可以调用该`.makeSound()`方法`goat`。

```js
const goat = {
  dietType: 'herbivore',
  makeSound() {
    console.log('baaa');
  },
  diet() {
    console.log(dietType);
  }
};
goat.diet();
// Output will be "ReferenceError: dietType is not defined"
```


```js
const goat = {
  dietType: 'herbivore',
  makeSound() {
    console.log('baaa');
  },
  diet() {
    console.log(this.dietType);
  }
};

goat.diet();
// Output: herbivore
```

该`this`关键字的引用调用对象提供访问调用对象的`属性`。在上面的例子中，调用对象`goat`，并通过使用`this`我们所访问的`goat`对象本身，然后`dietType`物业`goat`使用性质点符号。

## 3. Arrow Functions and this 箭头功能和 this
对于方法，调用对象是方法所属的对象。如果我们`this`在方法中使用关键字，那么值`this`是调用对象。但是，当我们开始使用方法的箭头函数时，它会变得有点复杂。看看下面的例子：

```js
const goat = {
  dietType: 'herbivore',
  makeSound() {
    console.log('baaa');
  },
  diet: () => {
    console.log(this.dietType);
  }
};

goat.diet(); // Prints undefined
// 上面把 `diet` 当初 `goat` 对象的属性,这个函数的啊this调用该对象的`dietType`,但是它的对象是 `diet`
```
- 引入箭头函数有两个方面的作用：更简短的函数并且不绑定`this`
- `箭头函数没有自己的this指针`<br>

在评论中，您可以看到它`goat.diet()`会记录`undefined`。所以发生了什么事？请注意，在`.diet()`使用箭头函数定义。

箭头函数固有地将已定义的this值绑定或绑定到不是调用对象的函数本身。在上面的代码片段中，值`this`是全局对象，或者是全局范围中存在的对象，它没有`dietType`属性，因此返回`undefined`。


```js
function Person() {
  // Person() 构造函数定义 `this`作为它自己的实例.
  this.age = 0;

  setInterval(function growUp() {
    // 在非严格模式, growUp()函数定义 `this`作为全局对象,
    // 与在 Person()构造函数中定义的 `this`并不相同.
    this.age++;
  }, 1000);
}

var p = new Person();
```

在ECMAScript 3/5中，通过将`this`值分配给封闭的变量，可以解决this问题。

```js
function Person() {
  var that = this;
  that.age = 0;

  setInterval(function growUp() {
    //  回调引用的是`that`变量, 其值是预期的对象.
    that.age++;
  }, 1000);
}
```

或者，可以创建绑定函数，以便将预先分配的`this`值传递到绑定的目标函数（上述示例中的`growUp()`函数）。

箭头函数不会创建自己的`this`,它只会从自己的作用域链的上一层继承`this`。因此，在下面的代码中，传递给`setInterval`的函数内的`this`与封闭函数中的`this`值相同：

```js
function Person(){
  this.age = 0;

  setInterval(() => {
    this.age++; // |this| 正确地指向 p 实例
  }, 1000);
}

var p = new Person();
```

## 4. Privacy 隐私
访问和更新属性是处理对象的基础。但是，在某些情况下，我们不希望其他代码只是访问和更新对象的属性。在讨论对象中的隐私时，我们将其定义为只有某些属性应该是可变的或能够在值上改变的想法。

某些语言具有内置于对象的隐私，但`JavaScript`没有此功能。相反，`JavaScript`开发人员遵循命名约定，向其他开发人员发出如何与属性进行交互的信号。一个常见的约定是`_`在属性名称前面加下划线表示`不应更改属性`。这是使用`_`前置属性的示例。

```js
const bankAccount = {
  _amount: 1000
}
```
在上面的示例中，`_amount`不打算直接操作。

即便如此，仍有可能重新分配`_amount`：

```js
bankAccount._amount = 1000000;
```
在后面的练习中，我们将介绍使用称为`getter`和`setter`的方法。这两种方法都用于`尊重预先`或`开始使用的属性`的意图_。`Getter`可以返回内部属性的值，`setter`可以安全地重新分配属性值。现在，让我们看看如果我们可以更改没有`setter`或`getter`的属性会发生什么。

## 5. Getters

`Getters`是获取和返回对象内部属性的方法。但他们可以做的不仅仅是检索财产的价值！我们来看看一个`getter`方法：

```js
const person = {
  _firstName: 'John',
  _lastName: 'Doe',
  get fullName() {
    if (this._firstName && this._lastName){
      return `${this._firstName} ${this._lastName}`;
    } else {
      return 'Missing a first name or a last name.';
    }
  }
}

// To call the getter method:
person.fullName; // 'John Doe'
```
请注意，在上面的`getter`方法中：

- 我们使用`get`关键字后跟一个函数。
- 我们使用的`if...else`条件，以检查是否都 `_firstName`和`_lastName`存在（通过确保他们都返回truthy值），然后返回根据结果不同的值。
- 我们可以使用调用对象的内部属性`this`。在fullName，我们正在访问`this._firstName`和`this._lastName`。
- 在最后一行，我们叫fullName上person。通常，不需要使用一组括号调用getter方法。从语法上讲，它看起来像是在访问一个属性。

现在我们已经讨论了语法，让我们讨论一下使用getter方法的一些显着优点：

- 获取房产时，`Getters`可以对数据执行操作。
- `Getter`可以使用条件返回不同的值。
- 在`getter`中，我们可以使用调用对象的属性`this`。
- 我们的代码功能更易于其他开发人员理解。

使用`getter`（和`setter`）方法时要记住的另一件事是属性不能与`getter / setter`函数共享同一名称。如果我们这样做，那么调用该方法将导致无限的调用堆栈错误。一种解决方法是在属性名称之前添加下划线，就像我们在上面的示例中所做的那样。

## 6. Setters

```js
const person = {
  _age: 37,
  set age(newAge){
    if (typeof newAge === 'number'){
      this._age = newAge;
    } else {
      console.log('You must assign a number to age');
    }
  }
};
```
请注意，在上面的示例中：

我们可以检查分配的值`this._age`。
当我们使用`setter`方法时，只会重新分配数字值 `this._age`
根据用于重新分配的值，有不同的输出`this._age`。
然后使用setter方法：

```js
person.age = 40;
console.log(person._age); // Logs: 40
person.age = '40'; // Logs: You must assign a number to age
```
`age`不需要使用一组括号调用`Setter`方法。在语法上，看起来我们正在重新分配属性的值。

与`getter`方法一样，使用`setter`方法也有类似的优点，包括检查输入，对属性执行操作以及显示对象应该如何使用的明确意图。尽管如此，即使使用`setter`方法，仍然可以直接重新分配属性。例如，在上面的示例中，我们仍然可以`._age`直接设置：

```js
person._age = 'forty-five'
console.log(person._age); // Prints forty-five
```

## 7. Factory Functions 工厂功能
到目前为止，我们一直在`单独`创建对象，但有时我们想要快速创建对象的许多实例。这就是`工厂`功能的用武之地。真实世界的工厂可以快速，大规模地生产多件物品。工厂函数是一个返回对象的函数，可以重复使用以生成`多个对象实例`。工厂函数也可以有参数，允许我们自定义返回的对象。

假设我们想创建一个在JavaScript中表示怪物的对象。有许多不同类型的怪物，我们可以单独制作每个怪物，但我们也可以使用工厂功能让我们的生活更轻松。为了实现创建多个怪物对象的恶魔计划，我们可以使用具有参数的工厂函数：

```js
const monsterFactory = (name, age, energySource, catchPhrase) => {
  return {
    name: name,
    age: age,
    energySource: energySource,
    scare() {
      console.log(catchPhrase);
    }
  }
};
```
在`monsterFactory`上面的函数，它有四个参数，并返回具有属性的对象`name`，`age`，`energySource`，和`scare()`。要创建一个代表像鬼一样的特定怪物的对象，我们可以调用m`onsterFactory`必要的参数并将返回值赋给变量：
```js
const ghost = monsterFactory('Ghouly', 251, 'ectoplasm', 'BOO!');
ghost.scare(); // 'BOO!'
```

## 8. Property Value Shorthand 属性值速记

`ES6`引入了一些新的快捷方式，用于将属性分配给称为解构的变量 。

在上一个练习中，我们创建了一个帮助我们创建对象的工厂函数。我们必须为每个属性分配一个键和值，即使键名与我们分配给它的参数名相同。提醒自己，这是工厂函数的截断版本：
```js
const monsterFactory = (name, age) => {
  return {
    name: name,
    age: age
  }
};
```
想象一下，如果我们必须包含更多属性，那么这个过程很快就会变得乏味！但我们可以使用称为属性值简写的解构技术来节省一些击键。下面的示例与上面的示例完全相同：

```js
const monsterFactory = (name, age) => {
  return {
    name,
    age
  }
};
```

## 9. Destructured Assignment 解构分配
我们经常希望从对象中提取`键值对`并将它们保存为属性。以下面的对象为例：

```js
const vampire = {
  name: 'Dracula',
  residence: 'Transylvania',
  preferences: {
    day: 'stay inside',
    night: 'satisfy appetite'
  }
};
```
如果我们想要将`residence`属性提取为变量，我们可以使用以下代码：

```js
const residence = vampire.residence;
console.log(residence); // Prints 'Transylvania'
```
但是，我们也可以利用称为`结构化任务`的解构技术来节省一些击键。在析构化赋值中，我们创建一个变量，其中包含一个对象密钥的名称，该密钥用大括号括起来`{ }`并为其`分配`对象。看看下面的例子：
```js
const { residence } = vampire;
console.log(residence); // Prints 'Transylvania'
```
`vampire`在第一个代码示例中回顾对象的属性。然后，在上面的例子中，我们声明一个新变量`residence`，它提取`residence`属性的值`vampire`。当我们记录`residence`控制台的值时，`'Transylvania'`会打印出来。

我们甚至可以使用析构化赋值来获取对象的嵌套属性：

```js
const { day } = vampire.preferences;
console.log(day); // Prints 'stay inside'
```

## 10. Built-in Object Methods 内置对象方法
在之前的练习中，我们一直在创建具有自己方法的对象实例。但是，我们也可以利用对象的内置方法！

例如，我们可以访问对象实例方法，例如：`.hasOwnProperty()`，`.valueOf()`等等！练习您的文档阅读技巧并查看：MDN的对象实例文档。

还有一些有用的对象类的方法，例如`Object.assign()`，`Object.entries()`和`Object.keys()`只是仅举几例。有关综合列表，请浏览：MDN的对象实例文档。

让我们熟悉一些这些方法及其文档。


# 总结
- 方法所属的对象称为调用对象。
- 该`this`关键字是指主叫对象，并且可以被用于调用对象的访问属性。
- 方法不会自动访问调用对象的其他内部属性。
- 值`this`取决于`this`访问的位置。
- 如果我们想要访问其他内部属性，我们不能使用箭头函数作为方法。
- `JavaScript`对象没有内置隐私，而是有一些约定可以通知其他开发人员有关代码的意图。
- 在属性名称之前使用下划线意味着原始开发人员不打算直接更改该属性。
- `Setter`和`getter`方法允许更详细的访问和分配属性的方法。
- `工厂函数`允许我们快速重复地创建对象实例。
- 有不同的方法来使用对象解构：一种方法是属性值的简写，另一种方式是解构分配。
