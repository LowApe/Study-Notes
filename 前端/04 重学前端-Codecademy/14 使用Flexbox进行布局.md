# 使用Flexbox进行布局
CSS提供了许多工具和属性，可用于在网页上定位元素。Codecademy关于盒子模型和CSS显示的课程介绍了其中的几种技术。

在本课程中，您将学习`flexbox`或`Flexible Box Layout`，这是一种为CS​​S3开发的新工具，它极大地简化了元素的定位方式。虽然`flexbox`并不意味着布局整个页面，但它对于定位元素非常有用，无论是单独还是成组。

`flexbox`布局有两个重要组件：flex容器和flex项。Flex容器是包含弹性项的页面上的元素。Flex容器的所有直接子元素都是flex项。这种区别很重要，因为您将在本课中学习的某些属性适用于Flex容器，而其他属性适用于Flex项目。

要将元素指定为`flex容器`，请将元素的`display`属性设置为`flex`或`inline-flex`。一旦一个项目是一个flex容器，我们就可以使用几个属性来指定它的子项的行为方式。在本课中，我们将介绍这些属性：

- justify-content
- align-items
- flex-grow
- flex-shrink
- flex-basis
- flex
- flex-wrap
- align-content
- flex-direction
- flex-flow
Flexbox是一款优雅的工具，可以轻松解决以前可能很难解决的定位问题。让我们开始吧！

## 1. display: flex
```css
div.container {
  display: flex;
}
```

## 2. inline-flex

在上一个练习中，您可能已经观察到，当我们给出`div`时 - 块级元素 - 它的`display`值`flex`仍然是块级元素。如果我们希望多个`flex`容器彼此内联显示怎么办？

如果我们不希望`div`元素是块级元素，我们会使用`display: inline`。但是，Flexbox `inline-flex`为`display`属性提供了值，这允许我们创建也是内联元素的`flex`容器。
```html
<div class="container">
  <p>I’m inside of a flex container!</p>
  <p>A flex container’s children are flex items!</p>
</div>
<div class="container">
  <p>I’m also a flex item!</p>
  <p>Me too!</p>
</div>
```

```css
.container {
  width: 200px;
  height: 200px;
  display: inline-flex;
}
```

在上面的示例中，有两个容器`div`。如果没有宽度，每个div都会拉伸页面的整个宽度。每个div中的段落也会显示在彼此之上，因为段落是块级元素。

当我们将`display`属性的值更改`inline-flex`为时，如果页面足够宽，则div将相互内联显示。在我们学习本课程的过程中，我们将更详细地介绍如何显示弹性项目。

请注意，在上面的示例中，设置了`Flex`容器的大小。目前，父容器的大小将覆盖其子元素的大小。如果父元素太小，则`flex`项将缩小以适应父容器的大小。我们将在以后的练习中解释原因。

```html


```

```css
.container {
  width: 200px;
}

.child {
  display: inline-flex;
  width: 150px;
  height: auto;
}
```
在上面的示例中，`.childdiv`将占用比`containerdiv`允许的更多宽度（300像素）（200像素）。该.childdiv一定收缩以适应容器的大小。在后面的练习中，我们将探索几种方法来处理这个问题。

## 2. justify-content
在之前的练习中，当我们将`display`父容器的值更改为`flex`或时`inline-flex`，所有子元素（flex项）都移向父容器的左上角。这是flex容器及其子容器的默认行为。我们可以指定弹性项目如何沿`主轴`从左向右展开。我们将在以后的练习中学习更多关于轴的知识。

要从左到右定位项目，我们使用一个名为的属性`justify-content`。
```css
.container {
  display: flex;
  justify-content: flex-end;
}
```

上面的示例中，我们将值设置`justify-content`为`flex-end`。这将导致所有弹性项目移动到弹性容器的`右侧`。

该justify-content属性有五个值：

1. flex-start - 所有项目将按顺序从父容器的左侧开始，在它们之间或之前没有额外的空间。
2. flex-end - 所有项目将按顺序放置，最后一项从父容器的右侧开始，在它们之间或之后没有额外的空间。
3. center - 所有项目将按顺序放置在父容器的中心，在它们之前，之间或之后没有额外空间。
4. space-around - 项目将在每个项目之前和之后以相等的空间定位，从而使元素之间的空间加倍。
5. space-between - 项目将在它们之间具有相等的空间，但在第一个元素之前或之后没有额外的空间。
在上面的定义中，“没有额外的空间”意味着将尊重边距和边框，但不会在元素之间添加更多的空间（比特定元素的样式规则中指定的空间）。此属性不会更改每个flex项的大小。


## 3. align-items
在上一个练习中，您学习了如何在页面的左侧到右侧证明Flex容器的内容。还可以在容器内垂直对齐柔性物品。该`align-items`属性可以`垂直空间`弹性项目。
```css
.container {
  align-items: baseline;
}
```

在上面的示例中，`align-items`属性设置为`baseline`。这意味着每个项目内容的`基线`将对齐。

我们可以为该align-items属性使用五个值：

- flex-start - 所有元素都将位于父容器的顶部。
- flex-end - 所有元素都将位于父容器的底部。
- center - 所有元素的中心将位于父容器的顶部和底部之间。
- baseline - 所有项目内容的底部将相互对齐。
- stretch - 如果可能，项目将从容器的顶部延伸到底部（这是默认值;具有指定高度的元素将不会拉伸;具有最小高度或未指定高度的元素将拉伸）。

这五个值告诉元素如何沿父容器的交叉轴行为。在这些示例中，横轴从容器的顶部到底部伸展。我们将在以后的练习中详细了解这一点。

你可能不熟悉的`min-height`和`max-height`属性，但你已经使用height和width之前。 `min-height`，`max-height`，`min-width`，和`max-width`是性质确保元件至少一定大小或至多一定规模。在本课程中，您将看到这些如何变得有用。

现在，您将看到上述五个值中的每个值！

## 4. flex-grow
在练习3中，我们了解到当flex容器太小时，所有flex项目按比例缩小。但是，如果父容器大于必要，则默认情况下不会伸展弹性项。该flex-grow属性允许我们指定项目是否应该增长以`填充`容器，以及哪些项目应该比其他项目成比例地增长。

```html
<div class="container">
  <div class="side">
    <h1>I’m on the side of the flex container!</h1>
  </div>
  <div class="center">
    <h1>I'm in the center of the flex container!</h1>
  </div>
  <div class=”side”>
    <h1>I'm on the other side of the flex container!</h1>
  </div>
</div>
```

```css
.container {
  display: flex;
}

.side {
  width: 100px;
  flex-grow: 1;
}

.center {
  width: 100px;
  flex-grow: 2;
}
```

在上面的示例中，.containerdiv的display值为flex，因此其三个子div将彼此相邻。如果.containerdiv中有额外的空间（在这种情况下，如果宽度大于300像素），弹性项目将增长以填充它。该.centerDIV将拉伸两倍的.side申报单。例如，如果有60个额外的空间像素，centerdiv将吸收30个像素，sidediv将每个吸收15个像素。

如果max-width为元素设置了a ，即使有更多的空间可以吸收，它也不会增大。

我们学习的所有以前的属性都是在flex容器或父元素上声明的。此属性 - flex-grow是我们在flex项目上声明的第一个属性。

> 自我理解:父容器特别小的时候flex会缩放比例,当特别大的时候,项目的大小(设置好的宽高)不会变化;flex-grow当我们拖动页面的情况下，会根绝我们设定的flex-grow值调整按比例拉伸弹性项目;
## 5. flex-shrink

正如`flex-grow`属性按比例拉伸弹性项目一样，该flex-shrink属性可用于指定哪些`元素`将缩小以及按比例缩放。

你可能已经在早期的练习中注意到，当flex容器太小时，flex项目会缩小，即使我们没有声明属性。这是因为默认值flex-shrink是1。然而，柔性项目不长，除非flex-grow变化，因为默认值flex-grow是0。

```html
<div class="container">
  <div class="side">
    <h1>I'm on the side of the flex container!</h1>
  </div>
  <div class="center">
    <h1>I'm in the center of the flex container!</h1>
  </div>
  <div class="side">
    <h1>I'm on the other side of the flex container!</h1>
  </div>
</div>
```

```css
.container {
  display: flex;
}

.side {
  width: 100px;
  flex-shrink: 1;
}

.center {
  width: 100px;
  flex-shrink: 2;
}   
```

在上面的例子中，如果div太小而不适合其中的元素，则`.center`将缩小两倍`.side`于`.container`。如果内容对于围绕它的Flex容器来说太大60像素，则`.center`将缩小30像素，外部div将缩小15像素。边距不受flex-grow和的影响flex-shrink。

请记住，最小和最大宽度优先于flex-grow和flex-shrink。与此一样flex-grow，flex-shrink只有在父容器太小或浏览器调整时才会使用。


> 自我理解:默认的 flex-shrink:1; 就是当页面小的情况下,缩放为1.0,此处设置的是缩放比例;flex-shrink:0 不进行缩放;

## 6. flex-basis

在前两个练习中，div的尺寸由CSS设置的高度和宽度决定。指定弹性项目宽度的另一种方法是使用`flex-basis`属性。`flex-basis`允许我们在项目`拉伸`或`缩小`之前指定项目的宽度。
```html
<div class="container">
  <div class=”side”>
    <h1>Left side!</h1>
  </div>
  <div class="center">
    <h1>Center!</h1>
  </div>
  <div class="side">
    <h1>Right side!</h1>
  </div>
</div>
```

```css
.container {
  display: flex;
}

.side {
  flex-grow: 1;
  flex-basis: 100px;
}

.center {
  flex-grow: 2;
  flex-basis: 150px;
}
```

在上面的示例中，`.side`的宽度为100像素，`.center`如果`.container`具有适当的空间（350像素，加上边距和边框的额外一点），则div将为150像素宽。如果·较大，.centerdiv将吸收两倍于`.side`的空间。

如果我们也将`flex-shrink`值分配给上面的div，情况也是如此。

> 自我总结: flex-basis 就是可以不用定义 width

# 7. flex

该flex属性提供了一种方便的方法来指定元素的拉伸和缩小方式，同时简化了所需的CSS。该flex特性允许你声明`flex-grow`，`flex-shrink`以及`flex-basis`所有在一行。

注意：该flex 属性与用于该属性的flex 值不同display。

```css
.big {
  flex-grow: 2;
  flex-shrink: 1;
  flex-basis: 150px;
}

.small {
  flex-grow: 1;
  flex-shrink: 2;
  flex-basis: 100px;
}
```

在上面的示例中，具有类的所有元素big将增加到具有类的元素的两倍small。请记住，这并不意味着big物品将是物品的两倍small，它们只占用更多的额外空间。

下面的CSS在一行中声明这三个属性。
```css
.big {
  flex: 2 1 150px;
}

.small {
  flex: 1 2 100px;
}
```
在上面的例子中，我们使用flex属性声明的值flex-grow，flex-shrink和flex-basis（按该顺序）所有在一行中。
```css
.big {
 flex: 2 1;
}
```
在上面的例子中，我们使用的flex财产申报`flex-grow`和`flex-shrink`，但不会flex-basis。

## 8. flex-wrap
有时，我们不希望我们的内容缩小以适应其容器。相反，我们可能希望flex项目在必要时移动到下一行。这可以使用flex-wrap属性声明。该flex-wrap属性可以接受三个值：

1. wrap - 不适合行的Flex容器的子元素将向下移动到下一行
2. wrap-reverse- 功能相同wrap，但Flex容器中行的顺序相反（例如，在2行flexbox中，wrap容器中的第一行将成为wrap-reverse第二行，wrap容器中的第二行将成为第一行） in wrap-reverse）
3. nowrap - 防止物品缠绕; 这是默认值，只需覆盖由不同CSS规则设置的包装值。

```html
<div class="container">
  <div class="item">
    <h1>We're going to wrap!</h1>
  </div>
  <div class="item">
    <h1>We're going to wrap!</h1>
  </div>
  <div class="item">
    <h1>We're going to wrap!</h1>
  </div>
</div>
```

```css
.container {
  display: inline-flex;
  flex-wrap: wrap;
  width: 250px;
}

.item {
  width: 100px;
  height: 100px;
}
```
在上面的示例中，父Flex容器包含三个flex项。Flex容器的宽度仅为250像素，因此三个100像素宽的柔性项目不能内嵌。该flex-wrap: wrap;设置使第三个溢出项目出现在另一个项目下方的新行上。

注意：该flex-wrap属性在Flex 容器上声明。

## 9. Align-content
现在元素可以换行到下一行，我们可能在同一个容器中有多行flex项。在上一个练习中，我们使用该`align-items`属性从Flex容器的顶部到底部空间展开项目。`align-items`用于对齐单行内的元素。如果Flex容器有多行内容，我们可以使用`align-content`从上到下的`行间隔`。

`align-content` 接受六个值：

1. `flex-start` - 所有元素行将位于父容器的顶部，两者之间没有额外的空间。
2. `flex-end` - 所有元素行都将位于父容器的底部，两者之间没有额外的空间。
3. `center` - 所有元素行都将位于父元素的中心，两者之间没有额外的空间。
4. `space-between` - 所有行元素将从容器的顶部到底部均匀间隔，在第一个上方或下方没有空间。
5. `space-around` - 所有元素行将从容器的顶部到底部均匀间隔，顶部和底部以及每个元素之间的空间量相同。
6. `stretch` - 如果指定了最小高度或没有高度，则元素行将拉伸以从上到下填充父容器（默认值）。

```html
<div class="container">
  <div class=”child”>
    <h1>1</h1>
  </div>
  <div class="child">
    <h1>2</h1>
  </div>
  <div class="child">
    <h1>3</h1>
  </div>
  <div class="child">
    <h1>4</h1>
  </div>
</div>
```

```css
.container {
  display: flex;
  width: 400px;
  height: 400px;
  flex-wrap: wrap;
  align-content: space-around;
}

.child {
  width: 150px;
  height: 150px;
}
```

在上面的示例中，Flex容器内有四个flex项。flex项目的宽度设置为`150`像素，但父容器的宽度仅为`400`像素。这意味着可以内联显示不超过两个元素。其他两个元素将换行到下一行，并且flex容器内部将有两行。该align-content属性设置为值space-around，这意味着两行div将从父容器的顶部到底部均匀地间隔开，在第一行之前和第二行之间具有相等的空格，行之间具有双倍空格。

## 10. flex-direction
到目前为止，我们只覆盖了水平拉伸和收缩并垂直包裹的柔性物品。如前所述，柔性容器具有两个轴：长轴和横轴。默认情况下，长轴为水平轴，横轴为垂直轴。

主轴用于定位具有以下属性的flex项：

1. `justify-content` 列位置
2. `flex-wrap` 换行
3. `flex-grow`
4. `flex-shrink`

横轴用于定位具有以下属性的flex项：

1. `align-items` 一行内
2. `align-content` 多行外
主轴和横轴是可互换的。我们可以使用该`flex-direction`属性切换它们。如果我们添加`flex-direction`属性并给它一个值`column`，则`flex`项将垂直排序，而不是水平排序。

```html
<div class="container">
  <div class="item">
    <h1>1</h1>
  </div>
  <div class="item">
    <h1>2</h1>
  </div>
  <div class="item">
    <h1>3</h1>
  </div>
  <div class="item">
    <h1>4</h1>
  </div>
  <div class="item">
    <h1>5</h1>
  </div>
</div>
```

```css
.container {
  display: flex;
  flex-direction: column;
  width: 1000px;
}
.item {
  height: 100px;
  width: 100px;
}
```

在上面的示例中，五个div将位于垂直列中。所有这些div都可以放在一个水平行中。但是，该`column`值告诉浏览器将div一个堆叠在另一个之上。如上所述，类似的属性`justify-content`将不会像以前的示例中那样表现。

该flex-direction属性可以接受四个值：

1. `row` - 元素将从左上角开始从左到右穿过父元素（默认）。
2. `row-reverse` - 元素将从右上角开始从右到左穿过父元素。
3. `column` - 元素将从左上角开始从父元素的顶部到底部定位。
4. `column-reverse` - 元素将从左下角开始从父元素的底部到顶部定位。

## 11. flex-flow
与flex属性一样，该flex-flow属性用于在一行中声明flex-wrap和flex-direction属性。

```css
.container {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
}
```

```css
.container {
  display: flex;
  flex-flow: column wrap;
}
```

在上面的示例中，`flex-flow`声明中的第一个`flex-direction`值是值，第二个`flex-wrap`值是值。对于所有的值`flex-direction`和`flex-wrap`被接受。


## 11. Nested Flexboxes 嵌套流布局
到目前为止，我们在同一页面上有多个flex容器来探索flex项目定位。还可以将柔性容器定位在彼此内部。

```html
<div class="container">
  <div class="left">
    <img class="small" src="#"/>
    <img class="small" src="#"/>
    <img class="small" src="#" />
  </div>
  <div class="right">
    <img class="big" src="#" />
  </div>
</div>
```

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.left {
  display: inline-flex;
  flex: 2 1 200px;
  flex-direction: column;
}

.right {
  display: inline-flex;
  flex: 1 2 400px;
  align-items: center;
}

.small {
  height: 200px;
  width: auto;
}

.large {
  height: 600px;
  width: auto;
}
```
在上面的示例中，具有三个较小图像的div将在页面左侧从上到下显示（.left）。还有一个div，其中一个大图像将显示在页面的右侧（.right）。左边的div有一个较小flex-basis但延伸以填充更多的额外空间; 正确的div有一个更大flex-basis但延伸，以填补更少的额外空间。两个div都是flex项和 flex容器。这些项具有指示它们将如何定位在父容器中的属性以及它们的flex项目子项将如何定位在它们中。

我们将使用上面相同的格式来将简单页面布局到右侧。


# 总结

1. display: flex 将元素更改为块级容器，其中包含flex项。
2. display: inline-flex 允许多个flex容器彼此串联显示。
3. justify-content 用于沿主轴间隔物品。
4. align-items 用于沿横轴间隔物品。
5. flex-grow 用于指定弹性项沿主轴吸收的空间（以及比例）。
6. flex-shrink 用于指定弹性项目缩小的程度以及沿主轴的比例。
7. flex-basis用于指定使用flex-grow和/或样式设置的元素的初始大小flex-shrink。
8. flex用于指定flex-grow,, flex-shrink和flex-basis在一个声明中。
9. flex-wrap 指定如果Flex容器不够大，元素应沿横轴移动。
10. align-content 用于沿横轴间隔行。
11. flex-direction 用于指定主轴和交叉轴。
12. flex-flow用于指定flex-wrap和flex-direction在一个声明中。
13. Flex容器可以通过声明display: flex或display: inline-flex为Flex容器的子项嵌套在彼此内部。
