# CSS 排版

## 1. font-family
在网页上设置字体时，请记住以下几点：

1. 样式表中指定的字体必须安装在用户的计算机上，以便在用户访问网页时显示该字体。我们将在稍后的练习中学习如何解决此问题。

2. 您可能已经注意到我们在本课程的前几个练习中没有指定字体。浏览器在显示网页时究竟知道使用哪种字体？所有大多数浏览器的默认字体是Times New Roman。如果您曾使用过格式化的文字处理器，则可能熟悉此字体。

3. 将网页上使用的字体数量限制为2或3是一种很好的做法。

4. 当字体的名称由多个单词组成时，必须用双引号括起来（否则将无法识别），如下所示：
```css
h1  { font-family：“Courier New” ; }
```

## 2. Font Weight

字体重量 三种方式：

```css
/* 粗体 */
p{
    font-weight:bold;
}
/* 非粗体 */
p{
    font-weight:normal;
}
/* 100~900 */
p{
    font-weight:400; // default
    font-weight:700; // bold
    font-weight:300; // light
    font-weight:400;
}
```

## 3. Font Style
```css
p{
    font-style: italic; // 斜体
}
p{
    font-style: normal; // 默认
}

p{
    font-style: oblique; //倾斜
}
```

## 4. Word Spacing

您还可以增加文本正文中单词之间的间距，技术上称为单词间距。

为此，您可以使用该`word-spacing`属性
```css
h1 {
    word-spacing: 0.3em;
}
```

## 5. Letter Spacing
您已经学会了如何增加文本和单词行之间的间距，但可以更加详细：增加单个字母之间的间距。

调整字母间距的技术术语称为“字距调整”。可以使用`letter-spacingCSS`中的属性调整字距。
```css
h1 {
    letter-spacing: 0.3em
}
```

## 6. Text Transformation 文字转换
文本也可以设置为使用text-transform属性以全部大写或小写形式显示。
```css
h1{
    text-transform: uppercase;// 大写
}
```

## 7. Text Alignment 字体对齐

该text-align属性可以设置为以下三个值之一：

1. `left` - 将文本对齐到浏览器的左侧。
2. `center` - 中心文本。
3. `right` - 将文本对齐到浏览器的右侧。

## 8. Line Height Anatomy 行高解剖

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0btn1bn7hj30qs057dgf.jpg)


## 9. Serif and Sans Serif

1. Serif - 在每个字母的末尾有额外细节的字体。例子包括Times New Roman或Georgia等字体。

2. Sans-Serif - 在每个字母的末尾没有额外细节的字体。相反，字母有直的扁平边缘，如Arial或Helvetica。

## 10. Fallback Fonts 后背字体
```css
h1 {
  font-family: "Garamond", "Times", serif;
}
```
上面的CSS规则说：

1. 将Garamond字体用于<h1>网页上的所有元素。
2. 如果没有Garamond，请使用Times字体。
3. 如果没有Garamond和Times，请使用用户计算机上预装的任何衬线字体。


## 11. Linking Fonts 链接字体

(https://fonts.google.com/)




1. 使用单个链接字体Droid Serif作为示例：

```html
<head>
  <link href="https://fonts.googleapis.com/css?family=Droid+Serif" type="text/css" rel="stylesheet">
</head>
```

2. 多个链接字体，使用Droid Serif和Playfair Display字体作为示例：

```html
<head>
  <link href="https://fonts.googleapis.com/css?family=Droid+Serif|Playfair+Display" type="text/css" rel="stylesheet">
</head>
```

3. 多种链接字体，以及重量和样式。这里Droid Serif拥有的字体粗细400，700和700i，同时Playfair Display具有的字体粗细400，700以及900i：

```html
<head>
  <link href="https://fonts.googleapis.com/css?family=Droid+Serif:400,700,700i|Playfair+Display:400,700,900i" rel="stylesheet">
</head>
```

## 12. Font-Face 自定义字体

还有其他方法可以链接不需要<link>在HTML文档中使用标记的非用户字体。CSS提供了一种将字体直接导入带有`@font-face`属性的样式表的方法。

要使用`@font-face`属性加载字体：

1. 不要使用`HTML`文档中的字体链接，而是在浏览器的`URL`栏中输入链接。
2. 浏览器将加载CSS规则。您需要关注直接标记为的规则`/* latin */`。一些拉丁规则是分开的。您将需要其中的每一个。
3. 复制标记为`latin`的每个CSS规则，并将规则从浏览器粘贴到`style.css`的顶部。

重要的是要强调需要将`@font-face`规则复制到样式表的顶部，以便在项目中正确加载字体。

方法：
1. `url:(https://fonts.googleapis.com/css?family=Droid+Serif|Playfair+Display)`
2. 复制到css 中

虽然Google字体和其他资源可以扩展字体选择，但您可能希望使用完全不同的字体或不使用外部服务中的字体。

我们也可以修改我们的@font-face规则以使用本地字体文件。我们可以为用户提供所需的字体系列，并将其与我们的网站一起托管，而不是依赖于不同的网站。

```css
@font-face {
  font-family: "Roboto";
  src: url(fonts/Roboto.woff2) format('woff2'),
       url(fonts/Roboto.woff) format('woff'),
       url(fonts/Roboto.tff) format('truetype');
}
```

在这里，您会注意到：

主要区别在于使用相对文件路径而不是`Web URL`。

我们为每个文件添加一种格式以指定要使用的字体。不同的浏览器支持不同的字体类型，因此提供多个字体文件选项将支持更多的浏

由于`.woff2`文件大小和性能的提高大大减少了，现在看来是未来的方式，但许多浏览器仍然不支持它。有许多很好的资源可以找到本地使用的字体，比如Font Squirrel(https://www.fontsquirrel.com/)。

# 总结

让我们回顾一下你到目前为止学到的东西：

- 排版是在页面上排列文本的艺术。

- 文本可以使用font-weight属性显示在任意数量的权重中。

- 文本可以使用font-style属性以斜体显示。

- 可以使用line-height属性修改文本行之间的垂直间距。

- Serif字体在每个字母的末尾都有额外的细节。Sans-Serif字体没有。

- 当用户的计算机上未安装某种字体时，将使用后备字体。

- Google字体提供可在带有<link>标记或@font-face属性的HTML文件中使用的免费字体。

- 可以使用@font-face属性和字体源的路径将本地字体添加到文档中。

- 该word-spacing属性改变了单个单词的距离。

- 该letter-spacing属性改变了单个字母之间的距离。

- 该text-align属性更改文本的水平对齐方式。
