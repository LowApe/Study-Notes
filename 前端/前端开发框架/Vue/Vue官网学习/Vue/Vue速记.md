# Vue速记

> Vue 使用速记



## Vue 基础用法



- 绑定属性`v-bind:key="value"`快捷别名 `:key="value"`
- 绑定事件`v-on`快捷别名`@事件="method"`
- 双向绑定`v-model="value"`
- 判断`v-if="boolean"`
- 循环`v-for=""`
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

