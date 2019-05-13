# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [插值](#插值)
- [数据绑定](#数据绑定)
- [样式绑定](#样式绑定)
- [过滤器](#过滤器)

<!-- /TOC -->
vue 基于 HTML 模板,响应式更新的机制,渐进式理念。


脚手架 vue-cli。因为它的存在,省去了手动配置开发环境、运行环境和测试环境的步骤。
安装好 npm,然后输入一下以下指令安装到机器的全局环境中:

```shell
npm install -g @vue/cli
npm i vue-cli -g
```
开始创建工程,键入一下的指令:
新建项目的基本问题回车跳过

```shell
vue init webpack filename
vue init webpack-simple vue-todos
vue create hello-world
```

安装完支持包后运行一个由脚手架构建的基本 Vue.js 程序:

```shell
npm run dev
npm run serve
```
> flag:一个是从 vue2实解密.一个是从极客时间的 Vue实战

```
|——README.MD
|——index.html  `#默认启动页面`
|——package.json `# npm 包配置文件`
|——src
|   |——App.vue      `#启动组件`
|   |——assets
|   |   |——logo.png
|   |__ main.js     `# Vue 实例启动入口`
|__ webpack.config.js   `# webpack 配置文件`
```

> flag：目前目录结构有点出入,可能书比较老了
先从 main.js 文件入手:

```js
import Vue from 'vue'
import App from './App.vue'

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
}).$mount('#app')

// ==========================

import Vue from 'vue'
import App from './App'

Vue.config.productionTip = false

new Vue({
  el: '#app',
  components: { App },
  template: '<App/>'
})
```

flag: `-_-` 又是两种APP 启动组件的模板
> 上一种 Vue2 新增 render 方法.Vue2 渲染机制与 React 一样。为了得到更好的运行速度,vue2 也采用了 Virtual DOM.
> Virtual DOM 是一种比游览器原生的 DOM 具有更好性能的虚拟组件模型

通过 import 将一个 Vue.js 的组件文件引入,并创建一个 Vue 对象的实例,在 Vue 实例中用 render 方法绘制 Vue 组件完成初始化。

然后,将 Vue 实例绑定到一个页面上,真实存在的元素 App Vue 程序就引导成功

打开 index.html 文件就能看到 Vue 实例与页面的对应关系:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <title>webshop</title>
  </head>
  <body>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>

```
> 也就是说,一个 **Vue 实例必须与一个页面元素绑定**.Vue实例一般用作 Vue 的全局配置使用,例如安装路由、资源插件,配置应用于全局的自定义过滤器、自定义指令等。

App.vue 这个文件，`*.vue` 是 Vue.js 特有的文件格式,表示的就是一个 Vue 组件，它也是 Vue.js 的最大特色,被称为**单页式组件**。`*.vue`文件可以同时承载"视图模式"、"样式定义" 和组件代码,它使的组件文件组织更加清晰与统一

Vue.js 的组件系统提供了一种抽象，让我们可以用独立可复用的小组件来构建大型应用。如果考虑这点，几乎任意类型的应用界面都可以抽象为一个组件树；

![](http://ww1.sinaimg.cn/large/006rAlqhly1g2x8vyv5zhj30p90dddhz.jpg)

```html
<template>
  <div id="app">
  </div>
</template>

<style></style>

<script>
export default {
  name: 'app',
}
</script>
```

由以上的代码我们可以了解到,单页组件由以下三个部分组件:

- `<template>`--视图模板;
- `<style>`--组件样式表;
- `<script>`--组件定义;

# 插值

Vue 的视图模板是基于 DOM 实现的。 这意味着所有的 Vue 模板都是可解析的有效的 HTML ,而且它对一些特殊的特性做了增强。接下来,我们就在模板上定义一个网页标题,并通过数据绑定语法将 App 组件上定义的数据模型绑定到模板上。

首先,在组件脚本定义中使用 data 定义用于内部访问的数据模型:

```js
export default {
  ...
  data(){
      return{
          title:"vue-todos"
      }
  }
}
```

data 可以是一个返回 Object 对象的函数,使用函数返回是为了可以具有更高的灵活性,例如对内部数据进行一些初始化的处理,官方推荐的用法是采用返回 Object 对象的函数。在模板中引用 data.title 数据时我们并不需要写上 data,这只是 Vue 定义时的一个内部数据容器,通过 Vue 的插值方式直接写上 title 即可:

```HTML
<h1>{{title}}</h1>
```

用双大括号 `{{}}` 引入的内容被称为 "Mustache" 语法,Mustache 标签会被相应数据对象的 title 属性的值替换。

插值是 Vue 模板语言的最基础用法,很多的变量输出都会采用插值的方式,而且插值还可以支持 JavaScript 表达式运算和过滤器。`{{}}`引用的内容都会被编码,如果要输出未被编码的文本,可以使用 `{{}}`对变量进行引用。

> 从 Vue 2 开始,组件模板必须且只能有一个顶层元素,如果在组件模块内设置多个顶层元素将被会引发编译异常。

template 属性是V，也就是视图,title 属性是 M,也就是模型,这个概念必须要了解


# 数据绑定

我们需要一个稍微复杂一点的数据模型来表述 Todo,它的结构应该是这样的:

```js
{
    value : '事项1', // 代办事项的文字内容
    done : false    // 标记事项是否已完成
}
```

因为是多个事项,那么这个**数据模型**应该是一个数组,为了能先显示这些待办事项,我们需要先设定一些样本数据,在 Vue 实例定义中的 data 属性中加入以下代码:

```js
export default{
    data(){
        return{
            title:'vue-Todos',
            todos:[
                {value:'读书1',done:false},
                {value:'读书2',done:true},
                {value:'读书3',done:false}
            ]
        }
    }
}
```
如果我们将 Vue 实例定义看作一个类的定义,data 相当于这个类的内容**字段属性**的定义区域。在 Vue 实例内的其他地方可以直接用 this 引用 data 内定义的任何属性,比如 this.title 就是就是引用了 data.title

> flag:现在版本好像不添加 this

我们要显示 todos 的数据就需要使用 Vue 模板的最常用的 v-for 指令标记,用来**枚举**数组并将对象渲染成一个列表。这个指令使用与 JS 类似的语法对 items 进行枚举,形式为 **item in items** ，items 是数据数组,item 是当前数组元素的别名:

```html
<ul>
    <li v-for="todo in todos">
        <label>{{todo.value}}</label>
    </li>
</ul>
```
如果要输出待办事项的序号,可以用 v-for 中隐藏的一个 index 值来进行输出,具体用法如下:

```html
<ul>
      <li v-for="(todo,index) in todos" :id="index" >
          <label>{{index+1}}.{{todo.value}}</label>
          <label>{{todo.done}}</label>
      </li>
</ul>
```
这个用法有点像 Python 的元组引用方式,只要用括号括住引用参数,最后一个值就是循环的索引。索引从0开始计算,所有我们 `+1` 从1开始,然后使用 JavaScript 的表达式插值来输出一个 index+1 的从1开始计数的序号。

这里除了用插值绑定，还使用了**属性绑定语法**,就是上面的 `:id="index"`，这样的写法是一种缩写,下图所示, 将 index 的值输出到 DOM 的 id 属性上。如果 index =1 ,那么输出结果就是 `id="1"`，如果没有在 id 前面加上 ":",那么 Vue 就会认为我们正在为 id 属性赋予一个字符串。
![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2xk04zinqj30ai03eglk.jpg)


> v-for 不单单可以循环渲染数组,还可以渲染对象属性

> flag:例子报错

# 样式绑定
由于 CSS 总是充满各种不得不重复的写法，所有我更愿意使用 less,以下是安装 webpack 支持 less 编译的包的方法:

```shell
npm i less style-loader css-loader less-loader -D
```

> 然后添加依赖,新版是自动加入

在 `/assets/` 中添加一个 todos.less 文件,并在 App.vue 的组件定义内引入 less 样式表:

```js
import './assets/todos.less'
```

使用 import 将样式表直接导入到代码的效果是:webpack 的less-loader 会生成一些代码,在页面运行的时候将编译后的 less 代码生成到 `<style>`标签内并自动插入到页面的 `<head>` 中.这种做法是全局的,在后面介绍路由部分时会有多个组件页面加载到同一页上,如果使用 import 导入样式的化,样式就会长期驻留页面直至 Vue 的根实例被销毁

> 通过 import 将样式文件导入是一种全局性的做法,也就是说,在每一个页面内的`<head>` 中都会有这一个样式表,这样做的缺点是很容易导致样式冲突。如果希望样式表仅应用于当前组件,可以使用 `<style scoped>`,然后用 CSS 的 `@import` 导入样式表:
```html
    <style scoped>
        /* @import './assets/todos.less' */
    </style>
```

前面只提到如何将 data 内定义的值以**文本插值**的方式输出到页面,并没有介绍如何将值"绑定"到属性内。样式的绑定和属性的绑定方式是一样的,我们这里就将 `done==true` 的代办事项`<li>` 绑定一个 checked 的样式类:

```html
<li v-for="(todo,index) in todos" :id="index" :class="{'checked':todo.done}">

</li>
```
**Vue 的属性绑定语法是通过 v-bind 实现的,完整的写法是这样:**

```html
<li v-for="(todo,index) in todos" :id="index" v-bind:class="{'checked':todo.done}">
</li>
```
但 v-bind 可以采用缩写方式":"表示,采用完整的写法又将会出现各种重复,建议使用缩写。Vue 的属性性绑定语法是 attribute="expression"，attribute 就是元素接受的属性值(既可以是原生的也可以是自定义的)expression 则是在 Vue 组件内由 data 或 props 内定义的对象属性,又或是一个合法的表达式。

> 注意:如果在元素属性中不加":"，Vue认为是向这个属性符上**字符串值**而不是 Vue 组件上定义的属性引用。

上例:class="{'checked':to.done}"的意思是:当 todo.done 为 true 时,向 `<li>` 元素的 class 添加 checked样式类。**凡是样式绑定必然是绑定到判断对象上的,不能直接写 CSS 类名,即使要绑定一个固定的 CSS 类也都是要这样写,':class="'btn':true"'**。

- `:class` 的 JSON 对象的值一定是布尔型,true 表示加上样式,false 表示移除样式类

- `:style` 的 JSON 对象则像是一个样式配置项,key 声明属性名,value 则是样式属性的具体值。

# 过滤器

代办事项的右侧增加一个时间字段 created ,并用 `<time>` 元素表示:

```HTML
<template>
  <div id="app">
    <h1>{{title}}</h1>
    <ul class="todos">
      <li v-for="(todo,index) in todos" :id="index" :class="{'checked':todo.done}">
          <label>{{index+1}}.{{todo.value}}</label>
          <label>{{todo.done}}</label>
          <time>{{todo.created | date}}</time>
      </li>
    </ul>
  </div>
</template>


<script>
// import HelloWorld from './components/HelloWorld'
import './assets/todos.less'

export default {
  name: 'App',
  data () {
      return{
        title:"待办清单",
        todos:[
          {value:"读书1",done:true,created:Date.now()},
          {value:"读书2",done:true,created:Date.now()+30000},
          {value:"读书3",done:false,created:Date.now()+30000000}
        ]
      }
  },
  components: {

  }
}
</script>
```

此时我们可以用一个很出名的时间格式化专用包 moment.js

```shell
npm i moment -S
cnpm i moment -S
```

引入 moment,并设定 moment 的区域为中国:

```html



<time>{{todo.created | date}</time>


<script>
import moment from 'moment'
import 'moment/locale/zh-cn'
moment.locale('zh-cn')
export default {
  name: 'app',
   data () {
      return{
        title:"待办清单",
        todos:[
          {value:"读书1",done:true,created: Date.now()},
          {value:"读书2",done:true,created:Date.now()+30000},
          {value:"读书3",done:false,created:Date.now()+30000000}
        ]
      }
  },
  filters:{
    date(val){
        return moment(val).calendar()
    }
  },
  components: {

  }
}
</script>
</script>
```

> ~~flag: 引用失败,不能格式化显示时间~~
失败原因: template 中应该 `{{}}` 而不是 `{{}`

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2zzlkc13bj313z0b4mxp.jpg)
