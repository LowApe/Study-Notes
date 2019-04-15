## 创建 XMLHttpRequest 对象

XMLHttpRequest 用于在后台与服务器交换数据。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

如果游览器版本不同的话
```js
var xmlhttp;
if(window.XMLHttpRequest){
 xmlhttp = new XMLHttpRequest
}else{
    //老版本 ie
    xmlhttp = new ActiveXObject("Microsoft.XMLHttpRequest")
}
```
## 向服务器发送请求

```js
xmlhttp.open("Get","test.txt",true);
xmlhttp.send();
```
|方法|参数|
|:---:|:---|
|open|menthod:请求的方式<br>url:文件在服务器上的位置<br>async:true(异步)或false(同步)|
|send|string:仅用于 POST 请求|

## POST 请求
|方法|参数|
|:---:|:---|
|setRequestHeader|header:规定头的名称<br>value:规定头的值|

## Async = true
当使用 async = true 时,请规定在响应处于 onreadystatechanges 事件中的就绪状态时执行的函数

```js
xmlhttp.onreadystatechange=function(){
    if(xmlhttp.readyState==4 && xmlhttp.staus==200){
        //操作
    }
}
xmlhttp.open("get","url",true);
xmlhttp.send();
```

## 服务器响应

|方法|参数|
|:---:|:---|
|responseText | 获取获取字符串形式的相应数据|
|resonseXML|获取 XML 形式的响应数据|

### responseText
```js
document.getElementByID("xxx").innerHTML=xmlHTML = xmlhttp.responseText;
```

### responseXML

```js
xmlDoc = xmlhttp.responseXML;
txt="";
x = xmlDoc.getElementsByTagName("ARTIST");
for (var i = 0; i < x.length; i++) {
    txt = txt + x[i].childNodes[0].nodeValue +"<br />";
}
document.getElementById("myDiv").innerHTML=txt;
```

## onreadystatechange 事件
请求被发送到服务器时，我们需要执行一些基于响应的任务。

每当 readyState 改变时，就会触发 onreadystatechange 事件。

readyState 属性存有 XMLHttpRequest 的状态信息。

|方法|参数|
|:---:|:---|
|readyState|存有 XMLHttpRequest 的状态, 从 0 到 4 发生变化<br><br> 0:请求未初始化<br> 1:服务器连接已建立<br> 2:请求已接收<br> 3:请求处理中<br> 4:请求已完成,且响应已就绪
|status| 200:"OK"<br> 404:未找到页面|

### 使用 Callback 函数
callback 函数是一种以参数形式传递给另一个函数的函数。
其实就是抽象出可以调用的函数
```js


function loadXMLDoc(url,cfunc)
{
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
xmlhttp.onreadystatechange=cfunc;
xmlhttp.open("GET",url,true);
xmlhttp.send();
}



function myFunction(){
loadXMLDoc("url",function(){
if(xmlhttp.readState==4&&xmlhttp.status==200){
    //操作
}
})
}
```
