
## 延迟加载
```js
<script src="example.js" defer></script>
```

> defer 属性当,html 页面加载之后在加载这条 js;

## 异步加载
由于游览器读取`<script>`是顺序读取,所有存在一种异步属性 `async`,当游览器下载了该js立即执行

```js
<script src="example.js" async></script>
```

# 总结

- HTML创建了网页的骨架，但JavaScript引入了交互性

- 该`<script>`元素有一个开始和结束标记。您可以在开始和结束`<script>`标记中嵌入JavaScript代码。

- 您可以使用开始标记中的`src`属性链接到外部JavaScript文件`<script>`。

- 默认情况下，只要HTML解析器在HTML文件中遇到脚本，脚本就会被加载并执行，从而阻止HTML解析器继续解析其余的页面元素。

- `defer`属性确保在执行脚本之前解析整个HTML文件。

- 异步标志将允许HTML解析器在下载脚本时继续解析，但在下载后立即执行。

# 文档对像模型DOM

![](http://ww1.sinaimg.cn/large/006rAlqhly1g1f28u8fpnj30st0f4ab2.jpg)

## 总结

- DOM是网页的结构模型，允许脚本语言访问该页面。
- DOM中的组织系统模仿HTML文档的嵌套结构。
- 嵌套在另一个元素中的元素称为该元素的子元素。它们嵌套在其中的元素称为这些元素的父元素。
- DOM还允许访问HTML元素的常规属性，例如它的样式，id等。

# Javascript和DOM

## 文档关键字 document
document 允许访问 DOM 的各个节点

## 调整元素

```js
.innerHTML  ='xxx'
```

`.innerHTML`重写替换元素,也可以添加子元素标签
```js
.innerHTML  ='<h1>xxx</h1>'
```

## 选择和修改元素
如果我们想要选择特定元素怎么办？DOM接口允许我们使用CSS选择器访问特定元素。    
该`.querySelector()`方法允许我们指定一个CSS选择器，然后返回匹配该选择器的第一个元素。
```js
document.querySelector('p');
```
您还可以使用其他`CSS`选择器，例如元素的`.`类或其`#ID`。


另一个选项，如果你想直接访问它们的元素id，你可以使用恰当命名的`.getElementByID()`函数：
```js
document.getElementByID('a').innerHTML='XXX'
```

## 修改元素样式
`.style`DOM元素的属性提供对该`HTML`标记的内联样式的访问。

语法遵循一种`element.style.property`格式，`property`表示`CSS`属性。

```js
let blueElement = document.querySelector('.blue');
blueElement.style.backgroundColor = 'blue';
```

与CSS不同，DOM样式属性不实现连字符，例如`background-colorcamel case`表示法`backgroundColor`。


## 创建和插入元素
正如DOM允许脚本修改现有元素一样，它也允许创建新元素。该`.createElement(tagName)`方法基于指定的标记名称创建新元素。
```js
let tags=document.createElement("li");
```
要创建元素并将其添加到网页，您必须将其指定为DOM上已存在的元素的子元素。我们称这个过程为追加。该`.appendChild()`方法将添加子元素作为最后一个子节点。

```js
xxx.appendChild(tagName);
```

## 删除元素

除了从头开始修改或创建元素之外，DOM还允许删除元素。该`.removeChild()`方法从父项中删除指定的子项。
```js
let paragraph = document.querySelector('p');
document.body.removeChild(paragraph);
```

如果要隐藏元素，因为它最初不需要加载，该`.hidden`属性允许您通过将其指定为`true`或`false`来隐藏它：

```js
document.getElementById('sign').hidden = true;
```

## 与 onclick 交互


```js
let element = document.querySelector("button");

function turnButtonRed (){
	element.style.backgroundColor = "red";
  element.style.color = "white";
  element.innerHTML = "Red Button";
}

element.onclick = turnButtonRed;
```

## 遍历DOM

每个DOM元素节点具有`.parentNode`和`.children`属性。该属性将返回元素子元素的列表，`null`如果元素没有子元素则返回

该`.firstChild`属性将授予对该父元素的第一个子元素的访问权限。

```js
let first = document.body.firstChild;
first.innerHTML = "First child";
first.parentNode.innerHTML = "I am the parent and my inner HTML has been replaced!";
```

# 总结
- 该document关键字允许访问JavaScript中的DOM的根
- DOM接口允许您使用该.querySelector()方法通过CSS选择器选择特定元素
- 您还可以通过其ID直接访问元素 .getElementById()
- 在.innerHTML和.style属性允许您通过分别改变其内容或风格来修改一个元素
- 您可以创建，追加和使用删除元素.createElement()，.appendChild()并.removeChild()分别方法
- 该.onclick属性可以基于click事件为DOM元素添加交互性

- 遍历 `.parentNode`, `.firstChild` ,`.children`
