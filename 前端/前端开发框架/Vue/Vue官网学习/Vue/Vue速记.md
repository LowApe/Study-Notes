# Vue速记

> Vue 使用速记



## Vue 基础用法



- `{{msg}}`插值可以接收property属性，变量变化了，插值也会发生变化
- `<span v-once>{{ span }}</span>`  `v-once`指令只能执行一次插槽值
- 插值只能是文本，如果插入html ，使用`<span v-hmtl="变量"></span>` 不推荐使用

- 绑定属性`v-bind:key="value"`快捷别名 `:key="value"`
- 绑定事件`v-on`快捷别名`@事件="method"`
	- 动态参数`v-bind:[变量或表达式]`
- 双向绑定`v-model="value"`
- 判断`v-if="boolean"`渲染元素到DOM，`v-show`始终渲染DOM上。**频繁切换使用 v-show**
- 循环`v-for=""`，`（value,name,index） in items` ，为每个项目添加key
- 引入组件`import`、`注册`、`使用`三部曲

## 组件实例 property



> 组件实例 property，所有都可以在组件的模版中访问,`$`以避免与用户定义的 property

- `data` property 定义初始化变量数据
- `props`组件接收数据
- `methods`组件内部方法
- `computed`
- `setup`
- `inject`
- `$emit`
- `$attrs`

## 生命周期钩子

- `created`用来在一个实例被创建之后执行代码
- 不要在选择 `property`或回调上使用`箭头函数`，**因为箭头函数并没有`this`**，比如`this`会作为变量一直向上级词法作用域查找，导致`ncaught TypeError: Cannot read property of undefined`

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gobrefaq06j31ao254gt1.jpg)

- 这就意味着只要 `message` 还没有发生改变，多次访问 `reversedMessage` 计算属性会立即返回之前的计算结果，而不必再次执行函数。





## Data Property 和方法

- 组件的`data()`选项是一个函数。Vue在创建组件实例的过程调用此函数。它返回一个对象，Vue通过响应式系统包裹成 `$data`的形式存储起来

- `data()`函数返回的对象，任何顶级 property 也能直接通过组件实例暴露出来。`vm.$data.key` 也可以`vm.key`

- ```js
	data(){
	    return {
	        a:4,
	    }
	}
	```

- Vue 自动`methods`绑定 `this`，以便它始终指向组件实例。这将确保方法在用作事件监听或回调时保持正确的`this`指向。

- 在定义`methods`时应避免使用箭头函数，因为这会阻止Vue绑定恰当的`this`指向

- `methods`和组件实例的其他所有 proerty 一样可以在组件的模版中被访问

- 

## 防抖和节流

- [防抖和节流](https://blog.csdn.net/weixin_39939012/article/details/101211869)

- [Lodash](https://www.lodashjs.com/docs/lodash.debounce#_debouncefunc-wait0-options)

	

# 计算属性和监听器

- 计算属性是基于它们的反应依赖关系缓存的，**计算属性只在相关响应式依赖发生改变时它们才会重新求值**
- 计算属性默认只有getter，只有在改变计算属性名称的时候,setter会被调用

- Vue 通过`watch`选项提供了一个更通用的方法，来响应数据的变化。**当需要数据变化时执行异步或开销较大的操作时，这个方式是最有用的**
- `v-bind`用于`class`和`sytle`时。**表达式结果的类型除了字符串之外，还可以是对象或数组**
- 





# 数组更新检测

## 变更方法

Vue 将被监听的数组的变更方法进行了包裹，所以它们也将会触发视图更新。

- `push()`添加元素
- `pop()`弹出元素
- `shift()`
- `unshift()`
- `splice()`
- `sort()`排序
- `reverse()`反转

## 替换数组

并不变更原始数组，总是返回一个新数组。

- `filter()`
- `concat()`
- `slice()`



