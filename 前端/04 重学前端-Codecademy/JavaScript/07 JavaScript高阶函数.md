# JavaScript高阶函数
我们经常不知道在用我们的母语与其他人交流时我们做出的假设数量。如果我们告诉你“数到三”，我们希望你能说出或想到数字一，二和三。我们假设你会知道从“一个”开始并以“三个”结束。通过编程，我们需要更明确地了解我们对计算机的指示。以下是我们如何告诉计算机“数到三”：

```js
for (let i = 1; i<=3; i++) {
  console.log(i)
}
```

我们还将学习另一种为编程添加抽象级别的方法：`高阶函数`。 高阶函数是接受其他函数作为参数和/或返回函数作为输出的函数。这使我们能够在其他抽象上构建抽象，就像“我们举办过生日派对”一样，抽象可以建立在抽象的“我们做了一个蛋糕”上。


## 1. Functions as Data 作为数据的功能
JavaScript函数的行为与语言中的任何其他数据类型相同; 我们可以为变量分配函数，我们可以将它们重新分配给新变量。

下面，我们有一个令人讨厌的长函数名称，它会损害使用它的任何代码的可读性。让我们假装这个功能做重要的工作，需要反复调用！
```js
const announceThatIAmDoingImportantWork = () => {
    console.log("I’m doing very important work!");
};

const busy = announceThatIAmDoingImportantWork;

busy(); // This function call barely takes any space!

```


`busy`是一个变量，它包含对原始函数的引用。如果我们可以在内存中查找地址busy并且内存中的地址`announceThatIAmDoingImportantWork`将指向同一个地方。我们的新`busy()`函数可以用括号调用，就好像这是我们最初赋予函数的名称。

注意我们如何在`announceThatIAmDoingImportantWork`没有括号的情况下作为`busy`变量赋值。我们想要分配函数本身的值，而不是它在调用时返回的值。

在`JavaScrip`t中，函数是第一类对象，这意味着像您遇到的其他对象一样，`JavaScript`函数可以具有属性和方法。

由于函数是一种对象，因此它们具有诸如`.length`和`.name`等方法之类的属性`.toString()`。您可以在文档中查看有关函数的方法和属性的更多信息 。


## 2. Functions as Parameters 作为参数的功能
由于函数的行为与JavaScript中的任何其他类型的数据一样，因此我们也可以将函数（转换为其他函数）作为参数传递给您。高阶函数是一个不接受函数作为参数的函数，返回一个功能或两个！我们将作为参数传入的函数调用并调用回调函数，因为它们在执行高阶函数时被调用。

当我们将函数作为参数传递给另一个函数时，我们不会调用它。调用该函数将计算该函数调用的返回值。使用回调函数，`我们通过键入不带括号的函数名来传递函数本身`（将评估调用函数的结果）：


```js
const timeFuncRuntime = funcParameter => {
   let t1 = Date.now();
   funcParameter();
   let t2 = Date.now();
   return t2 - t1;
}

const addOneToOne = () => 1 + 1;

timeFuncRuntime(addOneToOne);
```


```js
timeFuncRuntime(() => {
  for (let i = 10; i>0; i--){
    console.log(i);
  }
});
```

在这个例子中，我们timeFuncRuntime()使用一个从10开始向后计数的匿名函数调用。匿名函数也可以是参数！

# 总结

- 抽象允许我们以易于重用，调试和理解的方式编写复杂的代码供人类读者使用

- 我们可以像处理任何其他类型的数据一样处理函数，包括将它们重新分配给新变量

- `JavaScript`函数是第一类对象，因此它们具有像任何对象一样的属性和方法

- 函数可以作为参数传递给其他函数

- 高阶函数是接受函数作为参数，返回函数或两者兼有的函数
