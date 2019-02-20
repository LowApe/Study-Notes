# CSS 网格布局
在本课程中，我们将介绍一种名为CSS Grid的新功能强大的工具。该网格可以用来布局整个网页。尽管Flexbox主要用于在`一维布局`中定位项目，但CSS网格对于`二维布局`最有用，它提供了许多用于在行和列之间对齐和移动元素的工具。

## 1. 创建网格

要设置网格，您需要同时拥有网格容器和网格项。网格容器将是包含网格项作为子项的父元素，并对其应用总体样式和定位。

要将HTML元素转换为网格容器，必须将元素的`display`属性设置为`grid`（对于块级网格）或`inline-grid`（对于内联网格）。然后，您可以指定其他属性来布局网格。
```css
{
    display:grid;
}
```
## 2. 创建列

默认情况下，网格只包含一列。如果你要开始添加项目，每个项目将被放在一个新行上; 这不是一个网格！要更改此设置，我们需要明确定义网格中的行数和列数。

我们可以使用CSS属性定义网格的列`grid-template-columns`。以下是此属性的示例：

```css
.grid {
  display: grid;
  width: 500px;
  grid-template-columns: 100px 200px;
}
```

此属性创建两个更改。首先，它定义网格中的列数; 在这种情况下，有两个。其次，它设置每列的宽度。第一列为100像素宽，第二列为200像素宽。

我们还可以将列的大小定义为整个网格宽度的百分比。
```css
.grid {
  display: grid;
  width: 500px;
  grid-template-columns: 20% 200px 2px;
}
```

## 3. 创建行

我们已经学会了如何明确定义网格中的列数。要指定行的数量和大小，我们将使用该属性`grid-template-rows`。

这个属性几乎相同`grid-template-columns`。看看下面的代码，看看这两个属性在起作用。

```css
.grid {
  display: grid;
  width: 1000px;
  height: 500px;
  grid-template-columns: 100px 200px;
  grid-template-rows: 10% 20% 600px;
}
```

## 4. 网格模板

该属性`grid-template`可以替换前两个CSS属性。这两个grid-template-rows和grid-template-columns无处下面的代码被发现！

```css
.grid {
  display: grid;
  width: 1000px;
  height: 500px;
  grid-template: 200px 300px / 20% 10% 70%;
  /* 先行再列 */
}
```

## 5. Fraction 分数

您可能已经熟悉了几种类型的响应单元，例如百分比（`%`），`em`和`rem`。`CSS Grid`引入了一个新的相对大小单元 - `fr`就像分数一样。

通过使用`fr`单位，我们可以将列和行的大小定义为网格长度和宽度的一部分。该单元专门用于CSS Grid。使用`fr`可以更轻松地防止网格项溢出网格边界。请考虑以下代码：
```css
css .grid
{
    display: grid;
    width: 1000px;
    height: 400px;
    grid-template: 2fr 1fr 1fr / 1fr 3fr 1fr;
}
```


也可以fr与其他单位一起使用。发生这种情况时，每个fr代表可用空间的一小部分。

```css
css .grid {
    display: grid;
    width: 100px;
    grid-template-columns: 1fr 60px 1fr;
}
```

在该示例中，第二列占用60个像素。因此，第一列和第三列有40个可用于它们之间的分割。由于每个都占总数的一小部分，因此它们最终都是20像素宽。

## 6. Repeat 重复-函数

定义网格中行数和列数的属性可以将函数作为值。repeat()是这些功能之一。该repeat()功能专门为CSS Grid创建。

```css
.grid {
  display: grid;
  width: 300px;
  grid-template-columns: repeat(3, 100px);
}

/* The repeat function will duplicate the specifications for rows or columns a given number of times. In the example above, using the repeat function will make the grid have three columns that are each 100 pixels wide. It is the same as writing:

grid-template-columns: 100px 100px 100px; */

```

最后，第二个参数repeat()可以有多个值。
```css
{
    grid-template-columns: repeat(2, 20px 50px);
}
```

此代码将创建四列，其中第一列和第三列的宽度为20像素，第二列和第四列的宽度为50像素。

## 7. minmax 最小最大-函数
到目前为止，我们使用过的所有网格都是固定大小的。我们示例中的网格宽400像素，高500像素。但有时您可能希望根据Web浏览器的大小调整网格大小。

在这些情况下，您可能希望防止行或列变得太大或太小。例如，如果网格中有100像素宽的图像，则可能不希望其列比100像素更薄！该minmax()功能可以帮助我们解决这个问题。
```css
.grid {
  display: grid;
  grid-template-columns: 100px minmax(100px, 500px) 100px;
}

/* In this example, the first and third columns will always be 100 pixels wide, no matter the size of the grid. The second column, however, will vary in size as the overall grid resizes. The second column will always be between 100 and 500 pixels wide. */
```

## 8. Grid Gap 网格间距

到目前为止，在我们所有的网格中，网格中的项目之间没有任何空间。CSS属性grid-row-gap并将 grid-column-gap在网格中的每一行和列之间放置空格。

```css
css .grid {
    display: grid;
    width: 320px;
    grid-template-columns: repeat(3, 1fr);
    grid-column-gap: 10px;
    /* grid-gap: row column; */
}
```

# Grid Items 网格项目

## 1. Multiple Row Items 多行项目

使用CSS属性`grid-row-start`和`grid-row-end`，我们可以使单个网格项目占用多行。请记住，我们不再将CSS应用于外部网格容器; 我们正在为网格内的元素添加CSS！
```css
.item {
  grid-row-start: 1;
  grid-row-end: 3;
}
```

在此示例中，类的HTML元素item将占据网格中的两行，即行1和2.行`grid-row-start`和`grid-row-end`接受的值是网格线。

行网格线和列网格线从1开始，以比网格所具有的行数或列数大1的值结束。例如，如果网格有5行，则网格行的行范围为1到6.如果网格有8列，则网格行的行范围为1到9。


## 2. Grid Row and Column 网格行和列

我们可以使用该属性`grid-row`作为`grid-row-start`和的简写`grid-row-end`。以下两个代码块将生成相同的输出：
```css
.item {
  grid-row: 4 / 6;
}

.item {
  grid-column: 4 / span 2;
}
/* span 表示跨度 */
```
## 3. Grid Area 网格区域

`grid-area`采用斜杠分隔的四个值。订单很重要！这是如何`grid-area`解释这些值。

1. `grid-row-start`
2. `grid-column-start`
3. `grid-row-end`
4. `grid-column-end`

```css
.item {
  grid-area: 2 / 3 / 4 / span 5;
}
```

# 总结
- `grid-template-columns `定义网格列的数量和大小
- `grid-template-rows `定义网格行的数量和大小
- `grid-template`是用于定义两者`grid-template-columns`并`grid-template-rows`在一行中的简写
- `grid-gap` 在网格的行和/或列之间放置空格
- `grid-row-start`和`grid-row-end`使元件跨越栅格的某些行
- `grid-column-start`和`grid-column-end`使元件跨越栅格的某些列
- `grid-area`是的简写`grid-row-start`，`grid-column-start`，`grid-row-end`，和`grid-column-end`，都在一条线
