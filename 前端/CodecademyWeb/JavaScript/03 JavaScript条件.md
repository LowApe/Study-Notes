# JavaScript条件
- `if`，`else if`和`else`语句。
- 比较运算符。
- 逻辑运算符。
- truthy vs falsy values。
- 三元运算符。
- 该switch声明。

##  1. if else
```js
let sale=true;
if(sale){
    console.log('Time to Buy！');
}else{

}
```
## 2. 比较运算符
在编写条件语句时，有时我们需要使用不同类型的运算符来比较值。这些运算符称为比较运算符。

以下是一些方便的比较运算符及其语法的列表：

- 少于： `<`
- 比...更棒： `>`
- 小于或等于： `<=`
- 大于或等于： `>=`
- 等于： `===`
- 不等于： `!==`
比较运算符将左侧的值与右侧的值进行比较。例如：
```js
'apples'  ===  'oranges'  // false
```

## 3. Logical Operators 逻辑运算符

使用条件意味着我们将使用布尔值true或false值。在JavaScript中，有些运算符使用称为逻辑运算符的布尔值。我们可以使用逻辑运算符为条件添加更复杂的逻辑。有三个逻辑运算符：

- 与（&&）
- 或（||）
- 非（!）

## 4. 真假值

那么哪些值是假的 - 或者false在作为条件检查时评估？虚假值列表包括：

- `0`
- 像`""`或的空字符串`''`
- `null` 它代表什么时候没有价值
- `undefined` 表示声明的变量缺少值时
- `NaN`，或不是数字

```js
let numberOfApples = 0;

if (numberOfApples){
   console.log('Let us eat apples!');
} else {
   console.log('No apples left!');
}

// Prints 'No apples left!'
```

## 5. 短路分配
假设您有一个网站，并希望使用用户的用户名来制作个性化问候语。有时，用户没有帐户，使username变量麻痹。下面的代码检查是否username已定义，如果不是，则分配默认字符串：

```js
let defaultName;
if (username) {
  defaultName = username;
} else {
  defaultName = 'Stranger';
}
```
上面等价于下面

```js
let defaultName = username || 'Stranger';
```

## 6. Ternary Operator 三元运算符

```js
isNightTime ? console.log('Turn on the lights!') : console.log('Turn off the lights!');
```
# 总结

- `if`语句检查一个条件，如果该条件的值为true，则执行一个任务。
- `if……else`语句根据提供的条件做出二进制决策并执行不同的代码块。
- 我们可以使用`else if`语句添加更多条件。
- 比较运算符，包括`<`，`>`，`<=`，`>=`，`===`，`and !==`可以比较两个值。
- 逻辑和运算符`&&`或`“and”`检查所提供的两个表达式是否真实。
- 逻辑运算符`||`或`&&`检查所提供的表达式是否真实。
- 非操作符，!，转换一个值的真实性和虚假性。
- 三元运算符是一种简写法，用于简化以下情况:
其他语句。
- switch语句可以用来简化编写多个else if语句的过程。
- break关键字阻止在switch语句中检查和执行其余情况。
