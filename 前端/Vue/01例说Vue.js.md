# 目录

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
先从 min.js 文件入手:

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
> 也就是说,一个 Vue 实例必须与一个页面元素绑定.Vue实例一般用作 Vue 的全局配置使用,例如安装路由、资源插件,配置应用于全局的自定义过滤器、自定义指令等。

App.vue 这个文件，*.vue 是 Vue.js 特有的文件格式,表示的就是一个 Vue 组件，它也是 Vue.js 的最大特色,被称为**单页式组件**。"*.vue"文件可以同时承载"视图模式"、"样式定义" 和组件代码,它使的组件文件组织更加清晰与统一
