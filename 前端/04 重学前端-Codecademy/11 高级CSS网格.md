# 高级CSS网格
在上一课中，您学习了为网页创建`二维网格`布局所需的所有基础属性！在本课程中，您将学习以下可用于利用CSS网格布局功能的其他属性：

- grid-template-areas
- justify-items
- justify-content
- justify-self
- align-items
- align-content
- align-self
- grid-auto-rows
- grid-auto-columns
- grid-auto-flow
您还将了解显式和隐式网格和网格轴。

## 1. Grid Template Areas 网格模板区域

该grid-template-areas特性允许你命名你的网页的部分中要使用的值grid-row-start，grid-row-end，grid-col-start，grid-col-end，和grid-area属性。

```html
<div class="container">
  <header>Welcome!</header>
  <nav>Links!</nav>
  <section class="info">Info!</section>
  <section class="services">Services!</section>
  <footer>Contact us!</footer>
</div>
```

```css
.container {
  display: grid;
  max-width: 900px;
  position: relative;
  margin: auto;
  grid-template-areas: "head head"
                       "nav nav"
                       "info services"
                       "footer footer";
  grid-template-rows: 300px 120px 800px 120px;
  grid-template-columns: 1fr 3fr;
}

header {
  grid-area: head;
}

nav {
  grid-area: nav;
}

.info {
  grid-area: info;
}

.services {
  grid-area: services;
}

footer {
  grid-area: footer;
}
```

您可能希望展开网站的此部分以更清楚地查看上面的代码。

1. 在上面的示例中，HTML创建了一个包含五个不同部分的网页。
2. 规则集中的grid-template-areas声明.container创建一个2列，4行布局。
3. 该grid-template-rows声明指定每四行从上至下的高度：300个像素，120个像素，800个像素和120个像素。
4. 该grid-template-columns声明使用的fr值，使左边的列使用页面上可用空间右列的四分之一到页面上使用的可用空间的四分之三。
5. 在下面的每个规则集中.container，我们使用该grid-area属性来告诉该部分覆盖指定页面的部分。
> 该header元素跨越的第一行和两列。<br>
该nav元素跨越了第二排和两列。<br>
具有类的元素.info跨越第三行和左列。<br>
具有类的元素.services跨越第三行和右列。<br>
该footer元素跨越底行和两列。<br>


## 2. Overlapping Elements 重叠元素

CSS网格布局的另一个强大功能是能够轻松重叠元素。

重叠元素时，通常最简单的方法是使用网格线名称和`grid-area`属性

```html
<div class="container">
  <div class="info">Info!</div>
  <img src="#" />
  <div class="services">Services!</div>
</div>
```

```css
.container {
  display: grid;
  grid-template: repeat(8, 200px) / repeat(6, 100px);
}

.info {
  grid-area: 1 / 1 / 9 / 4;
}

.services {
  grid-area: 1 / 4 / 9 / 7;
}

img {
  grid-area: 2 / 3 / 5 / 5;
  z-index: 5;
}
```



在上面的示例中，有一个包含八行和六列的网格容器。容器中有三个网格项 - 一个<div>是类info，一个<div>是类services，一个是图像。

该info部分涵盖了所有八行和前三列。该services部分涵盖了所有八行和最后三列。

图像跨越第2行，第3行和第4行以及第3行和第4列。

z-index属性告诉浏览器将图像元素呈现在services和info部分的顶部，以使其可见。

## 3. Justify Items

在本课程中，我们多次提到“二维网格布局”。

网格布局中有两个轴 - 列（或块）轴和行（或内联）轴。

列轴在网页上从上到下延伸。

行轴在网页上从左向右延伸。

在以下四个练习中，我们将学习和使用依赖于理解网格轴的属性。

`justify-items`是一个沿着内联轴或行轴定位网格项的属性。这意味着它可以在网页上从左到右定位项目。

`justify-items` 接受这些值：

`start` - 将网格项对齐到网格区域的左侧
`end` - 将网格项对齐到网格区域的右侧
`center` - 将网格项对齐到网格区域的中心
`stretch` - 拉伸所有项目以填充网格区域

```html
<main>
  <div class="card">Card 1</div>
  <div class="card">Card 2</div>
  <div class="card">Card 3</div>
</main>
```

```css
main {
  display: grid;
  grid-template-columns: repeat(3, 400px);
  justify-items: center;
}
```

在上面的示例中，我们使用justify-items调整此网页上某些元素的位置。

1. 有一个网格容器，有三列，每列400像素宽。
2. 容器有三个没有指定宽度的网格项。
3. 如果不设置justify-items属性，这些元素将跨越它们所在列的宽度（400像素）。
4. 通过将justify-items属性设置为center，.card <div>s将在其列的内部居中。它们只会包含其内容所需的宽度（卡1等）。
5. 如果我们为.card元素指定宽度，它们将不会拉伸其列的宽度。

## 4. Justify Content

在上一个练习中，我们学习了如何在列中定位元素。在本练习中，我们将学习如何在其父元素中定位网格。

我们可以用来`justify-content`沿着行轴定位整个网格。

它接受这些值：

- start - 将网格对齐网格容器的左侧
- end - 将网格与网格容器的右侧对齐
- center - 将网格水平居中放置在网格容器中
- stretch - 拉伸网格项以增加网格的大小以在容器上水平扩展
- space-around - 在网格元素的每一侧包括相等数量的空间，导致元素之间的空间量增加一倍，因为在第一个元素之前和之后元素之前
- space-between - 在网格项之间包含相等的空间，两端都没有空格
- space-evenly - 在网格项目和两端之间放置均匀的空间


```html
<main>
  <div class="left">Left</div>
  <div class="right">Right</div>
</main>
```

```css
main {
  display: grid;
  width: 1000px;
  grid-template-columns: 300px 300px;
  grid-template-areas: "left right";
  justify-content: center; //上下居中也就是行
}
```

## 5. Align Items 对齐项目

在前两个练习中，我们学习了如何在页面上从左到右定位网格项和网格列。下面，我们将学习如何从上到下定位网格项目！

`align-items`是一个沿块或列轴定位网格项的属性。这意味着它可以从上到下定位项目。

- align-items 接受这些值：

- start - 将网格项对齐到网格区域的顶部
- end - 将网格项对齐到网格区域的底部
- center - 将网格项对齐到网格区域的中心
- stretch - 拉伸所有项目以填充网格区域

```html
<main>
  <div class="card">Card 1</div>
  <div class="card">Card 2</div>
  <div class="card">Card 3</div>
</main>
```

```css
main {
  display: grid;
  grid-template-rows: repeat(3, 400px);
  align-items: center;
}
```
在上面的示例中，我们使用`align-items`调整此网页上某些元素的位置。

1. 有一个网格容器，有三行，高400像素。
2. 容器有三个没有指定宽度的网格项。
3. 如果不设置`align-items`属性，这些元素将跨越它们所在行的高度（400像素）。
4. 通过将`align-items`属性设置`为center`，`.card` <div>将在其行的内部垂直居中。它们只会包含其内容所需的高度（单词卡1等）。
5. 如果我们为`.card`元素指定高度，即使`align-items: stretch;`已设置，它们也不会拉伸其行的高度。

## 6. Align Content 对齐内容

在上一个练习中，我们将网格项放在其行中。` align-content`沿着列轴或从上到下定位行。

它接受这些位置值：

- `start` - 将网格与网格容器的顶部对齐
- `end `- 将网格与网格容器的底部对齐
- `center` - 将网格垂直居中在网格容器中
- `stretch` - 拉伸网格项以增加网格的大小以在容器中垂直扩展
- `space-around` - 在网格元素的每一侧包括相等数量的空间，导致元素之间的空间量增加一倍，因为在第一个元素之前和之后元素之前
- `space-between` - 在网格项之间包含相等的空间，两端都没有空格
- `space-evenly `- 在网格项目和两端之间放置均匀的空间

```html
<main>
  <div class="top">Top</div>
  <div class="bottom">Bottom</div>
</main>
```

```css
main {
  display: grid;
  height: 600px;
  rows: 200px 200px;
  grid-template-areas: "top"
                       "bottom";
  align-content: center;
}
```

1. 在上面的示例中，网格容器的高度为600像素，但我们只指定了两行，每行200个像素。这将在网格容器中留下200像素的未使用空间。
2. `align-content: center;` 将行定位在网格的中心，在网格顶部留下100个像素，在网格底部留下100个像素。


## 7. Justify Self and Align Self

的`justify-items`和`align-items`属性指定包含在单个容器中的所有网格项目如何将分别自己定位沿行和列轴。

`justify-self`指定单个元素应如何相对于行轴定位自身。此属性将覆盖`justify-items`声明它的任何项目。

`align-self`指定单个元素应如何相对于列轴定位自身。此属性将覆盖`align-items`声明它的任何项目。

他们都接受这四个属性：

- start - 在网格区域的左侧/顶部定位网格项
- end - 在网格区域的右侧/底部定位网格项
- center - 将网格项目放置在网格区域的中心
- stretch - 定位网格项以填充网格区域（默认）

# Implicit vs. Explicit Grid 隐式与显式网格

到目前为止，我们已经使用各种属性明确定义了网格元素的维度和数量。这在许多情况下都很有效，例如企业的登录页面将始终显示特定数量的信息。

但是，有些情况下我们不知道我们要显示多少信息。例如，考虑在线购物。通常，这些网页包含搜索结果底部的选项，以显示一定数量的结果或在单个页面上显示所有结果。在显示所有结果时，Web开发人员无法预先知道每次搜索结果中将包含多少元素。

如果开发人员指定了3列5行网格（总共15个项目），但搜索结果返回30，会发生什么？

称为隐式网格的东西接管了。隐式网格是CSS Grid规范中内置的算法，当存在超过CSS指定的网格时，确定元素放置的默认行为。

隐式网格的默认行为如下：项目首先填充行，并根据需要添加新行。新的网格行只能高到足以包含其中的内容。在下一个练习中，您将学习如何更改此默认行为。

## 1. Grid Auto Rows and Grid Auto Columns 网格自动行和网格自动列

CSS Grid提供了两个属性来指定隐式添加的网格轨道的大小：`grid-auto-rows`和`grid-auto-columns`。

`grid-auto-rows`指定隐式添加的网格行的高度。 `grid-auto-columns`指定隐式添加的网格列的宽度。

`grid-auto-rows`并`grid-auto-columns`接受与其明确对应物相同的值，`grid-template-rows`并且`grid-template-columns`：

- 像素（px）
- 百分比（%）
- 分数（fr）
- 该`repeat()`功能

```html
<body>
  <div>Part 1</div>   
  <div>Part 2</div>
  <div>Part 3</div>
  <div>Part 4</div>
  <div>Part 5</div>
</body>
```

```css
body {
  display: grid;
  grid: repeat(2, 100px) / repeat(2, 150px);
  grid-auto-rows: 50px;
}
```

在上面的例子中，有5 <div>秒。但是，在`section`规则集中，我们只指定一个2行，2列网格 - 四个网格单元格。

第五个`<div>`将添加到一个50像素高的隐式行。

如果我们没有指定`grid-auto-rows`，则行将自动调整为网格项内容的高度。

## 2. Grid Auto Flow 网格自动流程

除了设置隐式添加的行和列的维度之外，我们还可以指定它们的呈现顺序。

`grid-auto-flow` 指定是否应将新元素添加到行或列。

`grid-auto-flow `接受这些值：

- row - 指定新元素应从左到右填充行，并在元素太多时创建新行（默认）
- column - 指定新元素应从上到下填充列，并在元素太多时创建新列
- dense - 如果添加了较小的元素，此关键字将调用一种算法，该算法会尝试在网格布局中填充漏洞
- 您可以配对row并column用dense，像这样：grid-auto-flow: row dense;。

# 总结

在使用CSS Grid创建布局时，您已经学习了许多新属性！让我们来复习：

- `grid-template-areas` 指定网格命名网格区域
- 网格布局是二维的：它们具有行或内联轴，以及列或块轴。
- `justify-items` 指定各个元素应如何在行轴上分布
- `justify-content` 指定元素组应如何跨行轴分布
- `justify-self` 指定单个元素应如何相对于行轴定位自身
- `align-items` 指定各个元素应如何跨列轴分布
- `align-content` 指定元素组应如何跨列轴分布
- `align-self` 指定单个元素相对于列轴的位置
----
- `grid-auto-rows` 指定隐式添加到网格的行的高度
- `grid-auto-columns` 指定隐式添加到网格的列的宽度
- `grid-auto-flow` 指定应在哪个方向创建隐式元素
