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


# 表单验证

有没有想过登录页面实际上如何运作？或者为什么用户名和密码的组合授予您访问网站的权限？答案在于验证。验证是根据所需数据检查用户提供的数据的概念。

有不同类型的验证。一种类型是服务器端验证，当数据被发送到另一台机器（通常是服务器）进行验证时会发生这种情况。此类验证的一个示例是使用登录页面。登录页面上的表单接受用户名和密码输入，然后将数据发送到服务器，该服务器检查该对是否正确匹配。

另一方面，如果我们想要检查浏览器（客户端）上的数据，我们使用客户端验证。在将数据发送到服务器之前进行此验证。不同的浏览器以不同的方式实现客户端验证，但它会导致相同的结果。

在不同的浏览器之间共享使用`HTML5`的内置客户端验证的好处。它节省了我们从必须向服务器发送信息并等待服务器发回确认或拒绝数据的时间。这还可以帮助我们保护我们的服务器免受来自恶意用户的恶意代码或数据的侵害。它还允许我们快速向用户提供特定字段的反馈，而不是让他们在输入表单的数据被拒绝时再次填写表单。

在本课中，我们将学习如何为我们的`<form>`添加一些验证检查！

## 需要的输入(Requiring an input)
有时我们的字段`<form>`不是可选的，即在我们提交之前必须提供信息。要强制执行此规则，我们可以将该`required`属性添加到`<input>`元素中。举个例子：

```HTML
<form action="/example.html" method="POST">
  <label for="allergies">Do you have any dietary restrictions?</label>
  <br>
  <input id="allergies" name="allergies" type="text" required>
  <br>
  <input type="submit" value="Submit">
</form>
```
这会呈现一个文本框，如果我们尝试提交`<form>`而不填写它，我们会收到以下消息：

弹出消息提示用户填写该字段

消息的样式因浏览器而异，上图描绘了`Chrome`浏览器中的消息。我们还会继续显示这些消息，因为它们会在以后的练习中显示在`Chrome`中。

让我们看看它是如何在您的浏览器中显示的！

## 设置最大值和最小值
我们可以使用的另一个内置验证是为数字字段分配最小值或最大值，例如`<input type="number">和<input type="range">`。要设置最小可接受值，我们使用该min属性并指定一个值。另一方面，为了设置最大可接受值，我们为max属性赋值。我们在代码中看到这个：

```HTML
<form action="/example.html" method="POST">
  <label for="guests">Enter # of guests:</label>
  <input id="guests" name="guests" type="number" min="1" max="4">
  <input type="submit" value="Submit">
</form>
```
如果用户尝试提交小于1的输入，则会显示警告： 在数字字段上提示用户输入大于或等于1的值

如果用户尝试输入大于4的数字，则会出现类似的消息。现在让我们看一下这个动作。

## 匹配模式(matching a pattern)

除了检查文本的长度之外，我们还可以添加验证来检查文本的提供方式。对于我们希望用户输入遵循特定指南的情况，我们使用该`pattern`属性并为其分配正则表达式或正则表达式。正则表达式是构成搜索模式的一系列字符。如果输入与正则表达式匹配，则可以提交表单。

假设我们想要检查有效的信用卡号码（14到16位数字）。我们可以使用正则表达式：`[0-9]{14,16}`它检查用户只提供了数字，并且他们输入了至少14位数，最多16位数。要将其添加到表单：

```HTML
<form action="/example.html" method="POST">
  <label for="payment">Credit Card Number (no spaces):</label>
  <br>
  <input id="payment" name="payment" type="text" required pattern="[0-9]{14,16}">
  <input type="submit" value="Submit">
</form>
```
使用`pattern`到位后，用户无法`<form>`使用不遵循正则表达式的数字提交。当他们尝试时，他们会看到如下的验证消息：

消息提示用户按照请求的格式

如果您想了解有关`Regex`的更多信息，请阅读[MDN的正则表达式文章](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions)。

要在实践中看到它，让`pattern`我们在HTML中使用该属性！

# 验证总结

让我们快速回顾一下：

- 在将信息发送到服务器之前，在浏览器中进行客户端验证。
- 将`required`属性添加到输入相关元素将验证输入字段中是否包含信息。
- `min`为数字输入元素的属性赋值将验证可接受的最小值。
- `max`为数字输入元素的属性赋值将验证可接受的最大值。
- `minlength`为文本输入元素的属性赋值将验证可接受的最小字符数。
- `maxlength`为文本输入元素的属性赋值将验证可接受的最大字符数。
- 分配正则表达式以`pattern`匹配提供的正则表达式的输入。
- 如果`<form>`未通过验证，则用户会收到解释原因和`<form>`无法提交的消息。
- 这些快速检查有助于确保输入数据对我们的服务器是正确和安全的。它还有助于为用户提供他们需要修复的内容的即时反馈，而不必等待服务器发回该信息。
