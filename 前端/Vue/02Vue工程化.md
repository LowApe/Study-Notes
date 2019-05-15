# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [脚手架 vue-cli](#脚手架-vue-cli)
	- [安装 vue-cli](#安装-vue-cli)
	- [使用 vue-cli 初始化项目](#使用-vue-cli-初始化项目)
	- [创建项目](#创建项目)
- [深入 vue-cli 的工程模板](#深入-vue-cli-的工程模板)
	- [webpack-simple 模板](#webpack-simple-模板)
	- [webpack 模板](#webpack-模板)
	- [构建工具](#构建工具)
		- [1.编译开发环境](#1编译开发环境)
			- [加载环境变量](#加载环境变量)
			- [合并 webpack 配置](#合并-webpack-配置)
			- [配置热加载](#配置热加载)
			- [配置代理服务器](#配置代理服务器)
			- [配置静态资源](#配置静态资源)
			- [加载开发服务器](#加载开发服务器)
		- [2. 编译生产环境](#2-编译生产环境)
- [Vue 工程的 webpack 配置与基本用法](#vue-工程的-webpack-配置与基本用法)
	- [webpack 的特点](#webpack-的特点)
		- [代码分割](#代码分割)
		- [加载器](#加载器)
		- [只能解析](#只能解析)
		- [插件系统](#插件系统)
	- [webpack 基本用法](#webpack-基本用法)
		- [样式表引用](#样式表引用)
		- [字体的引用](#字体的引用)
	- [用别名取代路径引用](#用别名取代路径引用)
	- [配置多个入口程序](#配置多个入口程序)
- [~~基于 Karma+Phantom+Mocha+Sinon+Chai 的单元测试环境~~](#基于-karmaphantommochasinonchai-的单元测试环境)
	- [Karma](#karma)
	- [PhantomJS](#phantomjs)
- [基于 Nightwatch 的端到端测试环境](#基于-nightwatch-的端到端测试环境)
	- [Nigthwatch](#nigthwatch)
	- [配置 Nitghtwatch](#配置-nitghtwatch)
	- [基本配置](#基本配置)
	- [Selenium 配置](#selenium-配置)
	- [测试环境配置](#测试环境配置)
	- [执行 E2E 测试](#执行-e2e-测试)
	- [使用无头游览器 PhantomJS](#使用无头游览器-phantomjs)
	- [Nihtwatch 与 Cucumber](#nihtwatch-与-cucumber)
- [相关链接](#相关链接)

<!-- /TOC -->
上一章节例子只能跑在**开发环境**,它的运行需要有 npm 和 npm 上的各种依赖包,所有我们没有办法直接将它放到某个服务器上就能运行。
本章介绍 Vue 如何为我们建立开发,测试与部署的自动化环境。
# 脚手架 vue-cli

vue-cli 类似于 java的 maven的包管理工具,前端开发时,不可避免要用到一大堆的工具,例如代码运行前的预处理器,程序热加载模块,代码校验,还有各种的测试环境与框架支持工具

## 安装 vue-cli

全局范围下安装
```shell
npm install vue-cli -g
```

## 使用 vue-cli 初始化项目

vue-cli 具体用法

用法: vue `<命令>` [参数]

命令:
- init  生成一个模板项目
- list  列出所有模板  
- build 原型文件项目
- create
- help

选项:
- `-h` 输出用法信息
- `-V` 输出版本号



```shell
vue list

  Available official templates:

  ★  browserify - A full-featured Browserify + vueify setup with hot-reload, linting & unit testing.
  ★  browserify-simple - A simple Browserify + vueify setup for quick prototyping.
  ★  pwa - PWA template for vue-cli based on the webpack template
  ★  simple - The simplest possible Vue setup in a single HTML file
  ★  webpack - A full-featured Webpack + vue-loader setup with hot reload, linting, testing & css extraction.
  ★  webpack-simple - A simple Webpack + vue-loader setup for quick prototyping.
```
browserfiy 比较老旧的构建工具,官方提供两个模板页出于对经常使用 browserify 的开发人员提供了一个熟悉的环境考虑。到正式项目开发,会走上 webpack 的道路

## 创建项目
init 命令,自动创建一个新的文件夹,并将所需的文件、目录、配置、
```shell
vue init webpack my-project
```
# 深入 vue-cli 的工程模板

webpack 和 webpack-simple 这两个模板从文件结构上几乎一直,一个是简化版,另一个是完全版。
- webpack-simple 是基于 Webpack@2.1.0-beta.25 进行配置的版本
- webpack 模板是基于 wbepack 1.3.2

两个版本暂时互不兼容,使用依赖包版本页不一样,所有不要将 webpack 模板创建的项目文件结构复制到 webpack-simple 中直接的取代升级,而是将 node_modules 内安装的所有的依赖包删除,然后重新安装才有可能迁移成功

## webpack-simple 模板
书上显示的工程目录结构(可能最新版本有变化):

```
|—— README.md
|—— index.md
|—— package.json
|—— src
|    |—— App.vue
|    |—— assets
|    |    |__logo.png
|    |____ main.js
|__ webpack.config.js
```
webpack-simple 只配置了 Babel 和 Vue 的编译器.src 目录,所有 Vue 代码源程序都放置在这个目录中,**五个模板构建出来的这个 src 目录都是一样的**,只是在 webpack 模板中多了 components 目录用于存放公用组件。

> 这个目录的结构与文件的组织应在开发前就进行约定,对于多人协作式项目,目录的使用与文件的命名都尤为重要。

**具体约定如下:**
1. 公用组件、指令、过滤器(多于三个文件以上的引用)将分别存放于 src 目录下的
    - components:
    - directives:
    - filters:
2. 以使用场景命名 Vue 的页面文件
3. 当页面文件具有私有组件、指令和过滤器时,则建立一个与页面同名的目录,页面文件更名为 index.vue,将页面与相关的依赖文件放在一起。
4. 目录由全小写的名词、动名词或分词命名,由两个以上的词组成,以 "-" 进行分割
5. Vue 文件同一以大驼峰命名法命名,仅入口文件 index.vue 采用小写。
6. 测试文件一律以测试,目标文件名`.spec.js`命名
7. 资源文件一律以小写字符命名,由两个以上的词组成用 "-" 进行分隔

> 前端开发的文件非常零碎而且会随着项目增多,组件化程度的增加使得文件越来越多,如果从一开始就没有约定目录和文件的使用与命名规范,项目越往后发展,寻找文件困难,所以一开始就确立工程的命名规范与使用约定是很有必要的。

## webpack 模板

```
|—— README.md
|—— build
|    |—— build.js
|    |—— check-versions.js
|    |—— untils.js
|    |—— vue-loader.js
|    |—— webpack.base.conf.js
|    |—— webpack.dev.conf.js
|    |—— webpack.prod.conf.js
|—— config
|    |—— dev.env.js
|    |—— index.js
|    |—— prod.env.js
|    |—— test.env.js
|—— package.json
|—— src
|    |—— App.vue
|    |—— assets
|    |    |__logo.png
|    |—— components
|    |    |__Hello.vue
|    |____ main.js
|—— static
|—— test
|    |—— e2e
|    |    |—— custom-assertions
|    |    |    |—— elementCount.js
|    |    |—— specs
|    |    |    |—— test.js
|    |    |—— nigthwatch.conf.js
|    |    |—— runner.js
|    |—— unit
|    |    |—— specs
|    |    |    |—— Hello.spec.js
|    |    |—— .eslintrc
|    |    |—— jest.conf.js
|    |    |—— setup.js

```

- build 存放用于编译用的 webpack 配置与相关的辅助工具代码
- connfig  存放三大环境配置文件,用于设定环境变量和必要的路径信息
- test 存放 E2E 测试与单元测试文件及相关配置文件
- static 存放项目所需要的其他静态资源文件
- dist 存放运行 npm run build 指令后的生产环境输出文件,可直接部署到服务器对应的静态资源文件内,该文件夹只有在运行 build 之后才会生成


## 构建工具
由于开发、测试、生产三大运行环境都需要进行构建,而且针对不同的环境要求,它的配置会有一定的区别,本书后面的章节中我们会对具体的配置进行一些定制与修改，我们应该清楚地了解 webpack 模板是如何进行构建的

### 1.编译开发环境
```shell
npm run dev
```

这个指令的配置是在 package.json 的 script 属性中设置的,实质上它是由 npm 来引导执行入口程序 webpack.dev.conf.js (以前版本是 dev-server.js) 完成以下的加载过程:

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2zws4dzfpj311f0fwdhh.jpg)

#### 加载环境变量
该环节从 config 目录加载 index.js 和 dev.env。js 两个模块,准备开发调试环境所必需的一些目录和全局变量

#### 合并 webpack 配置

在 build 目录下一共有三个 webpack 的配置文件:
- webpack.base.conf.js  公用的基本 webpack 配置
- webpack.dev.conf.js  开发环境专用的 webpack 配置;
- webpack.prod.conf.js 生产环境专用的 webpack 配置

这里使用了config 目录下的 dev.env.js 的 webpack-merge 的包来进行两个webpack 配置之间的合并,这个环节通过将 web.base.conf.js 和 webpack.conf.js 合并成最终的 webpack 配置


#### 配置热加载

上一个环境中合并的 webpack 配置也是通过这个环节被动态加载的,当代码文件发生变化,热加载就会启动 webpack 进行重新编译,然后将最新的编译文件重新加载到游览器中，

#### 配置代理服务器

这个环境是为我们代码增加一个模拟的服务端做准备,有了它的存在,我们就可以在没有后端程序支持的情况下,直接模拟远程服务器执行的一些请求效果。

> 例如,向服务器发出一个 HTTP GET/api/books/的请求,那么我们就可以利用代理服务器截获请求,然后返回一组这个 API 应该执行成功的返回结果,这一技术称为服务模拟

#### 配置静态资源

将图片、字体、样式表和编译后的 JS 脚本等,生产对应的一些印记(Footprint) 并存放到由开发服务器托管的一个 static 虚目录中,使得我们在游览器中可以正常访问到这些资源.

> flag:不理解——每个生成的文件Footprint是一些哈希代码，当文件内容发生变化时这些哈希代码就会发生改变，使用Footprint是将静态文件发布到CDN或者进行离线缓冲时通知浏览器文件是否发生改变的重要依据。

#### 加载开发服务器
启动一个 Express 的 web 服务器,将上述各个环境中配置好的模块进行加载,并使程序能通过游览器进行访问。

> 以上是 npm run dev 的完整执行思路

### 2. 编译生产环境

当项目准备发布时,在命令行键入:
```shell
npm run build
```
![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2zxnrej3rj30ji0awq40.jpg)

生产环境的构建过程比较简单,首先是对必要的资源文件进行打包加上 FootPrint.然后是对脚本进行编译、压缩和包大小的分割

# Vue 工程的 webpack 配置与基本用法
Vue 工程 需要对 webpack 配置进行修改,在此之前,我们需要对 webpack 有一个基本的认识,了解它到底能为我们做些什么。
webpack 是一个模块打包的工具,它的作用是把互相依赖的模块处理成静态资源。

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g2zws4dzfpj311f0fwdhh.jpg)

现有的模块打包工具不适合大型项目(大型 SPA——single page application 是一种网络应用程序模型)开发。当然最重要的还是因为而缺少代发分割功能,以及静态资源需要通过模块化来无缝衔接。webpack 目标:
- 把依赖树按需分割
- 把初始加载时间控制在较低的水平
- 每个静态资源都应该成为一个模块
- 能把第三方库集成到项目里成为一个模块
- 能定制模块打包器的每个部分
- 能适用于大型项目

## webpack 的特点

### 代码分割
在 webpack 的依赖树里有两种类型的依赖:同步依赖和异步依赖。异步依赖会成为一个代码分割点,并且组成一个心得代码块。 **在代码块组成的树被优化之后,每个代码块都会保存在一个单独的文件里**

> flag:不是很理解

### 加载器
webpack 原生是只能处理 JavaScript 的,而加载器的作用是把其他的代码转换成 JavaScirpt 代码,这样一来所有种类的代码都能组成一个模块,也就是说**通过 import 将 webpack 打包的资源以模块的方式引入到程序中**

以下是 Vue 项目中常用到的加载器(它们都是以 NPM 库形式提供的):
- vue-loader  用于加载与编译 `*.vue `文件
- vue-style-loader 用于加载 `*.vue` 文件中的样式
- style-loader 用于将样式直接插入到页面的 `<style>`页面
- css-loader 用于加载 `*.css` 样式表文件
- less-loader 用于编译与加载 `*.less`文件(需要依赖于less库)
- babel-loader 用于将 ES6 编译成为游览器兼容的 ES5
- file-loader 用于直接加载文件
- url-loader 用于加载 url 指定的文件,多用于字体与图片的加载
- json-loader 用于加载 `*.json` 文件为 JS 实例

### 只能解析

webpack 的智能解析器能处理几乎所有的第三方库,它甚至允许依赖出现这样的表达式:

```js
require("./components"+name+".vue")
```
它能处理大多数的模块系统,比如 CommonJS 和 AMD

> flag:AMD 是什么?

### 插件系统

## webpack 基本用法

### 样式表引用
某些页面或者组件可能具有特定的样式定义,这些样式对于其他页面来说是冗余的,我们只希望这些组件在应用时才自动加载这些特定的样式,此时用 webpack 我们就能在源代码中加入代码来动态加载 CSS
```html
<template>
  <div id="app">
  </div>
</template>
<script>

import './assets/todos.less'

export default {
    // ...
}
</script>

<style>
`#app` {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```

我们只需要在 webpack 配置中加入 less-loader，那么在打包的时候就会自动将 less 转换成 CSS ,并将 CSS 的动态代码生产到 JS 文件中。

### 字体的引用
假设在 dark.less 内加入对自定义字体文件的样式定义:

```less
@font-face{
    font-family: 'Darkenstone';
    src:url('./Darkenstone.eot');
    src:url('./Darkenstone.eot?#iefix');
    // 略,可以从 google font 下载

.header{
    display: flex;
    flex-flow: row nowrap;

    & > h1 {
        font:16pt 'Darkenstone';
    }
}
}
```
这里自定义字体游览器一定是不能识别的(本地),以前我们在样式表中定义这个字体样式并指定加载位置,很麻烦,如果使用 webpack 配置,在文件内加入一个 url-loader:

```js
{
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
```

> webpack 3.6 版本配置文件在 `config/webpack.base.conf.js`


webpack 在打包的时候为我们识别并在代码中引入字体的动态加载。

> vue-cli 的 webpack 模板已经为我们配置好了绝大多数常用的 loader,在实际运用中我们只需要了解它们是怎么来的,应该怎么用,需要时候如何修改就够了

## 用别名取代路径引用
在项目开发工程中有可能有许多包是没有放在 npm 上的,有一些教老的可能还依然只存在与 bower 上,某些甚至在 bower 与 npm 上都找不到,而不得不通过下载的方式在项目内引用,这样一来我们的代码可能通过 require 就得在代码内引用一段很长的文件路径,如下所示。

```js

import Selector from '../../bower_components/bootstrap-slect/dist/js/select.js'

```
这种包的引用方式明显违反了 CommonJS 的编程规范，对于这些长路径，甚至还具有"../.."这些相对路径搜索的定义，我们可以通过 webpack 的 resolve 配置项来解决。就以 select 这个组件为例，在 webpack.base.config.js 中加入以下的这个别名的定义:

```js
module.exports={
    entry:{...},
    output:{...},
    module:{...},
    resolve:{
        extensions:['','.js'],
        alias:{
            'bs_select':'bower_components/bootstrap-slect/dist/js/select.js'
        }
    }
}
```
有了这个别名定义,可以将上面那个长引用改为下面:
```js
import Selector from 'bs_select'
```
## 配置多个入口程序
多数情况下我们的程序入口不单单只有一个,举一个最简单的例子,前台提供给最终用户使用(http://www.domain.com/index),后台提供给登录用户使用(http://www.domain.com/admin),那么自然需要多个与 main.js 类似的程序入口。

首先在 `build/webpack.base.conf.js` 配置文件中的 entry 配置属性上加上新的入口文件:

```js
module.exports = {
    entry: {
        app: './src/main.js',
		admin: './src/admin-main.jss'
    },
}
```
这是用于告诉 webpack 哪几个是入口文件,这些文件需要被生成到启动页的 `<script>`内

`vue-cli` 的 webpack 模板使用 `HtmlWebpackPlugin` 插件，生成HTML入口页面并自动将生成后的JS文件和CSS文件的引用地址写入到页内的`<script>`中。

这里就需要在 `build/webpack.dev.conf.js`文件的 plugins 配置项内多配置一个 HtmlWebpackPlugin 插件，用于生成 admin.html 入口页

```js
plugins: [
// 省略
// 原有配置项m用于匹配注入 app.js 的输出脚本
new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
	  chunks: ['app'], // 与原配值得不同的是要用 chunks 指定对应的 entry
      inject: true
    }),
// 这是新增项,用于匹配注入 admin.js 的输出脚本
	new HtmlWebpackPlugin({
	      filename: 'admin.html',
	      template: 'index.html',
		  chunks: ['admin'], // 与原配值得不同的是要用 chunks 指定对应的 entry
	      inject: true
	    }),
```
还有就是得将同样的配置加入到生产环境专用的 webpack 配置文件 webpack.prod.conf.js 中,否则当我们运行 npm run build 是不会输出 admin.js 和 admin.html 这两个入口文件的

> 最后,如果使用了 vue-router 就得对 connect-history-api-fallback 插件的配置进行修改,否则原有的默认配置只会将所有的请求转发给 index.html,这样就会导致 History API 没有办法正确地将请求指向 admin.html ,导致热加载失败。新入口需要有明确区分的路由规则,否则还是会产生热加载失败的情况,这样就不便于开发了。

> flag: 不理解,而且代码文件版本不同,实际不是很好找

# ~~基于 Karma+Phantom+Mocha+Sinon+Chai 的单元测试环境~~

Vue 项目的单元测试用的最多的组件是功能测试,后面介绍如何写组件测试。先从开发流程的角度来看待测试的问题以及深入 vue-cli webpack 模板为我们建立的单元测试环境的功能与运作机理。

首先, 在有具体的组件测试目标时的 Vue-TDD 的开发流程如下

- 编写组件测试
- 编写组件引导
- 运行测试
- 重构

> 前端开发的单元测试环境会比后端开发的单元测试环境复杂,这是由于前端开发的工具碎片化比较严重所致的,所以要配置一个良好的单元环境需要有长期的实战经验以及熟悉各种各样的工具,vue-cli 的 webpack 模板给我们配置的单元测试环境其实也相当复杂,下图是这些工具是如何进行协作的。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g31rvuxheaj30l90dutau.jpg)

## Karma

Karam 是一个著名的测试加载器,它能完成许多测试环境加载任务
Karam 作为自动化测试程序的入口,它可以执行以下这些任务:
- 为测试程序注入指定依赖包
- 可同时在一个或多个游览器宿主中执行测试,满足兼容性测试需要
- 执行代码覆盖性测试
- 输出测试报告
- 执行自动化测试

Karam 就是这样一个开发环境,开发者指定需要测试的脚本/测试文件,需要运行的游览器信息,Karma 会在后台自动监控文件的修改,并启动一个游览器与 Karam 的服务器连接,这样当源代码或者测试发生修改后,Karma 会自动运行测试

开发者可以指定不同的游览器,甚至可以跨设备。由于 Karma 只是一个运行器,所有要配合一些测试框架如 Mocha、Jasmine 等作为单元测试的代码支持,甚至还可以自定义适配器来支持自己的测试框架。

Karma 拥有独立的 CLI ,可以通过以下方式安装到全局环境:
```shell
npm i karma  -g
```
然后就可以在命令行直接使用 Karma 指令了。

初始化 Karma 环境:

```shell
karma init
```
karma init 之路运行后就会出现一个向导型的终端交互界面来生成一个 karama.config.js 的全局配置文件:如果使用 vue-cli webpack 模板就不需要手动给,因为模板在创建工程时就生成了这个配置文件,在 `test/unnit/karma.conf.js`就能找到它

> flag:我看test 目录下是 e2e 我想测试工具不同。

```shell
karma start
```
这个命令就是启动 Karma ,让它按照 karma.conf.js 的配置项执行自动化测试。 vue-cli webpack 模板 将这个指令在 package.json 内进行了包装定义,所有在工程目录下只要运行:

```
npm run unit
```
直接启动 Karma

## PhantomJS

PhantomJS 是一个无界面的、可脚本编程的 WebKit 游览器引擎.它原生支持多种 Web 标准:DOM 操作、CSS选择器、JSON、Canvas以及SVG

一般来说,在我们的 Vue 单元测试代码中很少会直接以编码方式调用 PhantomJS的功能,更多是利用 PhantomJS 具有高速的运行速度这一特点,优化每一次单元测试的效能,节省等待游览器启动的漫长等待时间。

> flag:继续不下去了,版本不同...

# 基于 Nightwatch 的端到端测试环境

测试自动化意味着使用软件工具来反复运行项目中的测试,并为回归测试提供反馈

端到端测试简称为 E2E(End-TO-End)测试,它不同于单元测试侧重于检验函数的输出结果,端到端测试将**尽可能从用户的视角,对真实系统的访问行为进行仿真**。对于 Web 应用来说,这意味着需要打开游览器、加载页面、运行Javascript ,以及进行与 DOM 交互等操作。 **简而言之,单元测试的功能只能确保单个组件的质量,而 E2E 却正好面对组件与组件之间、用户与真实环境之间的一种集成性测试**

E2E 测试的意义在于可以通过程序固化和仿真用户操作,对于开发人员而言,基于 E2E 测试能极大地提高 Web 的开发效能,节约开发时间。

![](http://ww1.sinaimg.cn/large/006rAlqhly1g31tshx6xfj30ws0g8dmy.jpg)

从运行测试开始,所有的一切都是自动的！当我们写出 E2E 测试时就需要对操作需求有深刻的理解,这一过程还有机会对用户操作进行优化,从而提高用户体验。

## Nigthwatch

vue-cli 的 webpack 模板也为我们准备了一个当下很流行的 E2E 测试框架---Nightwatch

Nightwatch 是一套新近问世的基于 Node.js 的验收测试框架,使用 Slenium WebDriver API 以将 Web 应用测试自动化.它提供简单的语法,支持使用 JavaScript 和 CSS 选择器来编写运行在 Selenium 服务器上的端到端测试
工作流程:

![](http://ww1.sinaimg.cn/large/006rAlqhly1g31usrv3chj30l10r0gum.jpg)

Nightwatch 采用 Fluent interface 模式来简化端到端测试的编写,语法非常简介易懂:
```js
this.demoTestGoogle = function(browser){
	browser
		.url('http://www/google.com')


.waitForElementVisible('body',1000)
	.setValue('input[type=text]','neightwatch')
	.waitForElementVisible('buttom[name=btnG]',1000)
	.click('button[name=btnG]')
	.pause(1000)
	.assert.containsText('#main'，'The Night Watch')
	.end();
}
```
我们可以从 nightwatch 网站找到当前提供特性的列表:
- 简单但强大的语法.只需要使用 JavaScaript 和 CSS 选择器,开发者就能够非常迅速地撰写测试.开发者也不必初始化其他对象和类,只需要编写测试规范即可
- 内建命令行测试运行器,允许开发者同时运行全部测试 --分组或单个运行
- 自动管理 Selenium 服务器：如果 Selenium 运行在另一台机器上,那么也可以禁用此特性。
- 支持持续集成:内建 JUnit XML 报表,因此开发者可以在构建过程中,将自己得测试与系统(例如 Hudson 或 Teamcity等)集成
- 使用 CSS 选择器或 Xpath,定位并验证页面中的元素或是执行命令
- 易于扩展,便于开发者根据需要,实现与自己应用相关的命令

## 配置 Nitghtwatch

要了解 Nightwatch 的配置和用法,与前文介绍 Mocha 一样,应该先从工程结构

```
|__test
 	|__e2e
		|—— custom-assertions		 // 自定义断言
		|	|__ elementCount.js
		|—— page-objects			 // 页面对象文件夹
		|—— nightwatch.conf.js 		//nightwatch 运行配置
		|—— runner.js 				//运行器
		|__ specs					// 测试文件
				|__test.js
```

## 基本配置
Nightwatch 的配置项都集中在 nightwatch.conf.js 中,其实这个配置也可以是一个 JSON 格式,只需要简单地对配置项写入一些常量即可。但使用模块的方式进行配置可以执行一些额外的配置代码

Nightwatch 的配置分为以下三类:
- 基本配置:
- Selenium 配置:
- 测试环境配置

![](http://ww1.sinaimg.cn/large/006rAlqhly1g31vk8k7utj31280iw12w.jpg)

## Selenium 配置

下载 Selenium [http://selenium-release.storage.googleapis.com/index.html](http://selenium-release.storage.googleapis.com/index.html)

在 Vue 项目中如果使用 vue-cli,那么 Nightwatch 将不需要进行任何附加配置,否则你需要在命令还内安装 Selenium 的包装类库:
```
npm i selenium-server -D
```

Nightwatch 能引导 Selenium 的启动,在nightwatch.conf.js 配置文件中只需要声明 Selenium 服务器的二进制执行文件的具体路径即可,这个可以从 selenium-server 包提供的 Selenium 包装对象的path属性中获取,而无须将本机的物理路径写死到配置文件内。

var seleniumServer = require('selenium-server');

module.exports={
	"selenium": {
		"start_process": true,
		"server_path": seleniumServer.path,
		"port": 4444,
		"cli_args"：{
			"webdriver.chrome.driver": require('chromedriver').path
		}
	},
	// 省略
}

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g31wd9ney7j31430crgnw.jpg)

cli_args 的配置
- webdirver.firefox.profile：默认为每个会话创建一个独立的 FireFox 配置方案。如果你希望使用新的驱动配置可以在此进行声明。
- webdriver.chrome.driver：Nightwatch 同样可以使用 Chrome游览器加载测试,当然你要先下载一个 ChromeDriver 的二进制运行库对此进行支持。此配置项用于指明 ChromeDriver 的安装位置。

## 测试环境配置

test_settings 内的项目将应用于所有的测试实例,在E2E 测试中我们可以通过Nightwatch 默认实例对象 browser 获取这些配置,`vue-cli` 为我们创建了 default、firefox和chrome 三个环境配置项,default 配置是应用与所有环境的基础配置选项,其他的配置项会自动覆盖default

```js
test_settings: {
    default: {
      selenium_port: 4444,
      selenium_host: 'localhost',
      silent: true,
      globals: {
        devServerURL: 'http://localhost:' + (process.env.PORT || config.dev.port)
      }
    },

    chrome: {
      desiredCapabilities: {
        browserName: 'chrome',
        javascriptEnabled: true,
        acceptSslCerts: true
      }
    },

    firefox: {
      desiredCapabilities: {
        browserName: 'firefox',
        javascriptEnabled: true,
        acceptSslCerts: true
      }
    }
}
```

> flag:没有具体操作

## 执行 E2E 测试

```shell
npm run e2e --[环境]
```

## 使用无头游览器 PhantomJS

我们的开发环境在配置 Mocha 和 Karma 时就已经安装了 PhantomJS ，但如果你细读 Nightwatch(nightwatch.conf.js) 的默认配置会惊奇地发现根本没有采用PhantomJS，只是配置了Chrome和Firefox,因为慢！Chrome 的启动是很慢的,所以我们会选择 PhantomJS,但是没有默认配置 PhantomJS,需要我们手动配置

```js
test_settings: {
    default: {
      selenium_port: 4444,
      selenium_host: 'localhost',
      silent: true,
      globals: {
        devServerURL: 'http://localhost:' + (process.env.PORT || config.dev.port)
      }
    },

    chrome: {
      desiredCapabilities: {
        browserName: 'chrome',
        javascriptEnabled: true,
        acceptSslCerts: true
      }
    },
	phantom: {
		desiredCapabilities: {
			browserName:'phantomjs',
			javascriptEnabled:true,
			acceptSslCerts:true,
			phantomjs.page.settings.userAgent: 'Mozilla/5.0(Macintosh;IntelMacOSX10_10_5)AppleWebKit/537.36(KHTML,likeGecko)Chrome/46.0.2490.80Safari/537.36',
			phantomjs.binary.path:'node_modules/phantomjs-prebuilt/bin/phantomjs'
		}
	}
}
```

## Nihtwatch 与 Cucumber



# 相关链接
- [https://www.webpackjs.com/](https://www.webpackjs.com/)
