# JavaScript概述

> 绝大多数现代网站都使用了 JavaScript，并且所有的现代 Web 游览器均包含了 **JavaScript解释器**。
>
> JavaScript 版本：ECMAScript 6 简称 ES6
>
> Google JavaScript 解释器：简称 V8 3.0
>
> JavaScript 语言 **核心针对文本、数组、日期和正则表达式的操作定义了API**，这些 API 不包括输入和输出功能。

# JavaScript 语言核心

- 快速概览

## 定义变量和类型

```javascript
// 双横线注释内容
var x; // 声明一个变量x
x=0; // = 赋值给x0

x=1;
x=0.01;// 整数和实数公用一种数据类型
x="hello"; // 双引号内的文本构成的字符串
x='JavaScript'; // 单引号内的文本同样构成字符串
x=true; // 布尔值
x=false; // 另一个布尔值
x=null; // null 是一个特殊的值
x=undefined; // 未定义
```

## 对象

```javascript
// JavaScript 中最重要的类型就是对象
var book={
    key:"value",
    key2:true
}
// 通过"."或"[]"来访问对象属性
book.key; // "value"
book["key2"]; // true
book.author = "author"; // 赋值一个新的属性
book.contents={}; // {}是一个空对象，它没有属性
```

## 数组

```javascript
// JavaScript 同样支持数组(以数字为索引)
var primes=[2,3,5,6];
primes[0] // 索引为0 的元素 2
primes.length // 4
primes[4]=9; // 新索引则赋值新的元素
primes[4]=11; // 赋值

var empty=[]; // 定义一个空数组

// 数组和对象中都可以包含数组或对象
var points=[ // 具有两个元素的数组
    {x:0,y:0},// 每个元素都是一个对象
    {x:1,y:1}
];

var data={// 一个包含两个属性的对象
    data1:[[1,2],[3,4]],
    data2:[[2,3],[4,5]]
}
```

## 运算符

```javascript
3+2 // 5
3-2 // 1
3*2 // 6
3/2 // 1.5
var ponints=[
    {x:0,y:0},
    {x:1,y:1}
]
points[1].x-pints[0].x // 1 - 0 = 1
"3"+"2" // "32" 字符串加法运算也可以作字符串连接
var count = 0; // 定义一个变量
count++; // 1
count--; // 0
count+=2;// 2
count*=3;// 6

// 相等关系
x==y
x!=y
x<y
x<=y
x>y
"two"=="three"
"two">"three"
false==(x>y) 

// 逻辑运算符是对布尔值的合并或求反
(x==2)&&(y==3)
(x>3)||(y<3)
!(x==y)  
```

## 函数

```javascript
// 返回一个比传入值大1的值
function plus1(x){
    return x+1;
};
plus1(1);
// 函数是一种值可以赋值给变量
var square=function(x){
    return x*x;
};
```

## 方法

**当将函数和对象合写在一起，函数就变成了方法**

```javascript
// 函数赋值给对象的属性，称为方法
var a=[];
a.push(1,2,3); // push() 方法向数组中添加元素,push方法如下 dist方法一下
// 这里 push 和 reverse 都是 javascirpt 内置的数组方法
a.reverse(); // 另一个方法:将数组元素的次序反转

points.dist = function(){
    var p1 = this[0];  //  this 获取当前数组
    var p2 = this[1];   
    var a = p2.x-p1.x; 
    var b = p2.y-p1.y; 
    return Math.sqrt(a*a+b*b);
};
points.dist();
```



## 控制语句

```javascript
// 绝对值
function abs(x){
    if(x>=0){
        return x;
    }else{
        return -x;
    }
}
```

## 面向对象的编程

```javascript
// 定义一个构造函数以初始化一个新的 Point 对象
function Point(x,y){ // 构造函数均以大写字母开始
    this.x = x; // 将函数参数存储为对象的属性
    this.y = y;
}
// new 关键字和构造函数来创建一个示例
var p = new Point(1,1);

// 通过给构造函数的 prototype 对象赋值

Point.prototype.r=function(){
    return Math.sqrt(this.x*this.x + this.y*this.y);
}; // Point 的示例对象 p 继承了方法r()
p.r();
```

# JavaScript 客户端

## 嵌入 HTML 文件中

JavaScript 代码可以通过`<script>`标签来嵌入到 HTML 文件中

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        //嵌入 html文件
    </script>
</body>
</html>
```

## 全局函数

```html
<script>
    // 弹出对话框，确定时，url 变为 http://taobao.com
    function moveon(){
        var answer = confirm("准备好了吗");
        if(answer)
            window.location = "http://taobao.com"
    } 
    // 1000ms 后调用 moveon 函数
    setTimeout(moveon,1000);
</script>
```

## 通过脚本操控 HTML 文档内容

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        function debug(msg){
            var log = document.getElementById("debuglog");// 获取 id 是 debuglog 节点元素
            if(!log){//校验存在性
                log = document.createElement("div"); // 创建 div 节点元素
                log.id = "debuglog"; // 节点属性 id = "debuglog"
                log.innerHTML = "<h1>Degbug Log</h1>";// log 内容为 Degbug Log
                document.body.appendChild(log); // 将创建的节点添加到 body 末尾
            }
            // 将消息包装在 <pre> 中，并添加至 log 中
            var pre =  document.createElement("pre");
            var text = document.createTextNode(msg); // 将 msg 包装在一个文本节点中
            pre.appendChild(text);// pre 添加这个 text 消息节点

            log.appendChild(pre); // 
        }
        debug("Change World!");
        
    </script>
</body>
</html>
```

## 通过脚本操控 CSS 样式

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1 id = "world" class="world">Change World</h1>
    <script>
        
        function hide(e,reflow){
            if(reflow){
                e.style.display = "none"; // 隐藏这个元素,所占空间消失
            }else{
                e.style.visibility = "hidden"; // 隐藏，但是保留所占空间
            }
        }

        function highlight(e){
            if(!e.className){
                e.className = "hilite";
            }else{
                e.className += "hilite";
            }
        }
        // hide(document.getElementById("world"),false);
        highlight(document.getElementById("world"));
    </script>
</body>
</html>
```



## 事件处理程序操控

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="js/debug.js"></script>
    <script src="js/hide.js"></script>
</head>
<body>
    Hello
    <button onclick="hide(this,true);degbug('hide button 1');">Hide1</button>
    <button onclick="hide(this);degbug('hide button 2');">Hide2</button>
    World
</body>
</html>
```

"load" 事件注册一个事件处理程序。同时，也展示了注册"click"事件处理函数

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        // "load" 事件只有在文档加载完成后才会触发
        window.onload = function(){
            // 获取所有 img 标签元素数组
            var images = document.getElementsByTagName("img");

            for(var i = 0;i<images.length;i++){
                var image = images[i];
                if(image.addEventListener){ 
                    // 注册了点击事件处理程序
                    image.addEventListener("click",hide,false);
                }else{
                    // 兼容 IE8
                    image.attachEvent("onclock",hide);
                }
            }
        }
        // 事件处理函数
        function hide(event){
            event.target.style.visibility = "hidden";
        }
    </script>
</body>
</html>
```

## JQuery 库



## 示例：贷款计算机 Web 应用