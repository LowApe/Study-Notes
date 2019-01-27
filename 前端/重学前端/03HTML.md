# HTML 标签
您现在已经了解了构建HTML页面和添加不同类型内容所需的所有基本元素和设置。在CSS的帮助下，您很快就会创建漂亮的网站！

虽然某些标签具有非常特定的用途，例如图像和视频标签，但大多数标签用于描述它们所包含的内容，这有助于我们稍后修改和设置内容的样式。似乎有无数的标签可供使用（比我们教过的要多得多）。知道何时使用每个是基于您想要如何描述HTML的内容。描述性的，精心挑选的标签是高质量Web开发的关键。可以在[Mozilla](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element)文档中找到可用HTML标记的完整列表。

让我们回顾一下您在本课中学到的内容：

1. 该`<!DOCTYPE html>`声明应该总是出现的代码在你的`HTML`文件的第一行。这使浏览器可以知道期望的HTML版本。
2. 该`<html>`元素将包含您的所有`HTML`代码。
3. 有关网页的信息（如标题）属于`<head>`页面内容。
4. 您可以使用`<title>`头部内的元素为网页添加标题。
5. 网页标题显示在浏览器的标签中。
6. `Anchor tags（<a>）`用于链接到同一页面上的内部页面，外部页面或内容。
7. 您可以在网页上创建部分并使用`<a>`标记跳转到它们并将ids 添加到要跳转到的元素。
8. HTML元素之间的空格有助于使代码更易于阅读，同时不会更改元素在浏览器中的显示方式。
9. 缩进还有助于使代码更易于阅读。它使父子关系可见。
10. 注释使用以下语法以HTML编写：`<!-- comment -->`。

# HTML 表格

1. 该`<table>`元素创建一个表。
2. 该`<tr`>元素向表中添加行。
3. 要将数据添加到行，可以使用该`<td>`元素。
4. 表格标题阐明了数据的含义。标题添加`<th>`元素。
5. 表数据可以使用该`colspan`属性跨越列。
6. 表数据可以使用该`rowspan`属性跨行。
7. 表可以分为三个主要部分：头部，主体和页脚。
8. 使用`<thead>`元素创建表头。
9. 使用`<tbody>`元素创建表的主体。
10. 使用`<tfoot>`元素创建表的页脚。

# HTML 表单

## label
在上一个练习中，我们创建了一个`<input>`元素，但是我们没有包含任何内容来解释`<input>`它的用途。为了让用户正确识别`<input>`我们使用适当命名的`<label>`元素。

该`<label>`元素具有开始和结束标记，并显示在开始和结束标记之间写入的文本。要关联`<label>`和`<input>`，`<input>`需要`id`属性。然后，我们指定`for`的属性`<label>`与价值要素id的属性`<input>`，就像这样：

```HTML
<form action="/example.html" method="POST">
  <label for="meal">What do you want to eat?</label>
  <br>
  <input type="text" name="food" id="meal">
</form>

```

看，现在用户知道该`<input>`元素的用途！使用该`<label>`元素的另一个好处是，当单击此元素时，将`<input>`突出显示/选择相应的元素。


## number

我们现在已经过了两个与文本相关的type属性`<input>`。但是，我们可能希望我们的用户键入一个数字 - 在这种情况下，我们可以将type属性设置为...（您猜对了）...... "number"！

通过设置type="number"一个`<input>`我们可以限制哪些用户输入到输入字段只是数字（以及一些特殊字符，如-，+和.）。我们还可以提供一个step属性，该属性在输入字段内创建箭头，以通过step属性的值增加或减少。下面是为数字呈现输入字段所需的代码：

```HTML
<form>
  <label for="years"> Years of experience: </label>
  <input id="years" name="years" type="number" step="1">
</form>
```
哪个呈现：

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1fzl5hoyr6gj30fj022jrc.jpg)

渲染数字输入字段，箭头指向字段的右侧

现在是时候应用这些知识了。


## 范围输入(range input)

`<input type="number">` 如果我们想让用户输入他们选择的任意数量，那么使用a 非常棒。但是，如果我们想限制用户可以键入的数字，我们可能会考虑使用不同的`type`值。我们可以使用的另一个选项是设置`type`为`"range"`创建滑块。

要设置滑块的最小值和最大值，我们赋值给`min`和`max`属性`<input>`。我们还可以通过为`step`属性赋值来控制滑块的平滑度和流畅度。较小的`step`值将使滑块更加流畅，而较大的step值将使滑块更明显地移动。看一下代码来创建一个滑块：

```HTML
<form>
  <label for="volume"> Volume Control</label>
  <input id="volume" name="volume" type="range" min="0" max="100" step="1">
</form>
```
上面的代码呈现：

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1fzl5kdz23cj30aj01rjr8.jpg)

用于音量控制的渲染滑块

在上面的示例中，每次滑块移动一次时，`<input>'s value`属性的值都会更改。

## 下拉列表(DropDown list)

如果我们希望我们的用户从几个可见选项中选择一个选项，单选按钮是很棒的，但想象一下我们是否有一个完整的选项列表！这种情况很快就会导致很多单选按钮被跟踪。

另一种解决方案是使用下拉列表允许我们的用户从有组织的列表中选择一个选项。这是创建下拉菜单的代码：

```HTML
<form>
  <label for="lunch">What's for lunch?</label>
  <select id="lunch" name="lunch">
    <option value="pizza">Pizza</option>
    <option value="curry">Curry</option>
    <option value="salad">Salad</option>
    <option value="ramen">Ramen</option>
    <option value="tacos">Tacos</option>
  </select>
</form>
```
哪个呈现： 渲染下拉列表，显示第一个选项

如果我们点击包含第一个选项的字段，则会显示以下列表： 渲染下拉列表，显示所有选项

请注意我们在代码中使用元素`<select>`来创建下拉列表。要填充下拉列表，我们添加多个`<option>`元素，每个元素都有一个`value`属性。默认情况下，只能选择其中一个选项。

呈现的文本是开始和结束`<option>`标记之间包含的文本。但是，它是提交中`value`使用的属性的值`<form>`（注意文本和`value`大小写的差异）。当`<form>`被提交时，从该输入字段中的信息将使用被发送`name`的`<select>`和`value`所选择的`<option>`。例如，如果用户从下拉列表中选择了`Pizza`，则信息将作为`"lunch=pizza"`。


## 数据输入(datalist input)

即使我们有一个有组织的下拉列表，如果列表有很多选项，用户滚动整个列表以找到一个选项可能会很乏味。这就是使用`<datalist>`元素派上用场的地方。

它`<datalist>`与`<input type="text">`元素一起使用。将`<input>`创建一个文本字段，用户可以键入从进入和过滤选项`<datalist>`。让我们来看一个具体的例子：

```HTML
<form>
  <label for="city">Ideal city to visit?</label>
  <input type="text" list="cities" id="city" name="city">

  <datalist id="cities">
    <option value="New York City"></option>
    <option value="Tokyo"></option>
    <option value="Barcelona"></option>
    <option value="Mexico City"></option>
    <option value="Melbourne"></option>
    <option value="Other"></option>  
  </datalist>
</form>
```
请注意，在上面的代码中，我们有`<input>`一个`list`属性。该`<input>`关联到`<datalist>`通过`<input>`的`list`属性和`id`的`<datalist>`。

虽然`<select>`并且`<datalist>`有一些相似之处，但存在一些重大差异。在关联`<input>`元素中，用户可以键入输入字段以搜索特定选项。如果`<option>`中没有一个匹配，则用户仍然可以使用他们输入的内容。当提交表单时，所选的选项`<input>`的`name`和值以及`value`用户输入的值将作为一对发送。

## textarea
一个`<input`>元素，用于type="text"创建单行输入字段，供用户输入信息。但是，有些用户需要编写更多信息，例如博客文章。在这种情况下，`<input>`我们可以使用，而不是使用`<textarea>`。

该`<textarea>`元素用于创建更大的文本字段，供用户编写更多文本。我们可以添加属性`rows`并`cols`确定行的数量和列数`<textarea>`。看一看：

```html
<form>
  <label for="blog">New Blog Post: </label>
  <br>
  <textarea id="blog" name="blog" rows="5" cols="30">
  </textarea>
</form>
```
在上面的代码中，一个`<textarea>`5行乘30列的空白呈现给页面，如下所示：



如果我们想要一个更大的文本字段，我们可以单击并拖动右下角来展开它。

当我们提交表单时，值`<textarea>`是在框内写的文本。如果我们想在文本中添加默认值，`<textarea>`我们会将其包含在开始和结束标记中，如下所示：

```HTML
<textarea>Adding default text</textarea>
```
此代码将呈现`<textarea>`包含预填充文本的内容：“添加默认文本”。

但是，不要只听我们的话，让我们测试一下吧！

# 表单总结

很好的工作与极其常见和有用的`<form>`元素互动！

在本课中我们讨论了：

- 的目的`<form>`是允许用户输入信息并发送。
- 该`<form>`的`action`属性决定了形式的信息去。
- 所述`<form>`的`method`属性确定信息是如何发送和处理。
- 要为用户添加输入信息的字段，我们使用`<input>`元素并将type属性设置为我们选择的字段：
- 设置type以"text"创建文本输入单行场。
- 设置type以"password"创建用于检查文本输入的单个行字段。
- 设置type以"number"创建数字输入单行场。
- 设置type以"range"创建滑块以从一系列数字中进行选择。
- 设置type为"checkbox"创建一个可与其他复选框配对的复选框。
- 设置type以"radio"创建可与其他单选按钮配对的单选按钮。
- 设置type为"list"将`<input>`与`<datalist>`元素配对。
- 设置type以`"submit"`创建提交按钮。
- 甲`<select>`元件被填充有`<option>`元件，并呈现一个下拉列表选择。
- 一个`<datalist>`元素填充`<option>`元素以及与工程`<input>`通过选择搜索。
- 一个`<textarea>`元素是有一个可定制区域的文本输入字段。
- `<form>`提交时`name`，接受输入的字段和`value`这些字段中的字段`name=value`将成对发送。
- 将`<form>`元素与上面列出的其他元素结合使用，可以创建考虑用户需求的网站。抓住机会，学习所学知识并应用它！
