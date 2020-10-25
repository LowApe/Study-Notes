# React组件的生命周期及其使用场景
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0m6fzj3bbj30wv0lgwgg.jpg)
[React生命周期方法图示网址](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

创建时：
- constructor 组件构建
- **getDeriverdStateFromProps 用于外部属性初始化内部状态**
- render 描述UI DOM 结构的的地方，必须定义
- componentDidMount() ，这里可以发 ajax 请求，定义外部作用

更新时:
- 新的属性传入 New props 新的组件传入
- 内部修改状态 setState()
- forceUpdate() 调用就强制更新
- getDeriverdStateFromProps 外部属性更新内部状态
- **shouldComponentUpdate() 告诉组件是否 render**如果 props 变化 ui 并不需要变化，直接返回
- getShapeshotBeforeUpadte() 
- componentDidUpdate()

卸载时:
当界面消失:
- componentWillUnmount 做一些资源释放

----
# 详细说明
## constructor

1. 用于初始化内部状态,很少使用
2. 唯一可以修改 state 的地方(平常修改用 setState)

```js
this.state.xxx= xxx
```

----
## getDerivedStateFromProps()

> 外部属性来初始化内部状态的方法

1. 当 state 需要从 props 初始化时使用
2. 尽量不要使用: 维护两者状态一致性会增加复杂度
3. 每次 render 都会调用
4. 经典场景:表单控件获取默认值

----
## componentDidMount

> 组件作为唯一一次的事情在这里设置

1. UI 渲染完成后调用-安全操作 DOM 或请求 AJAX
2. 只执行一次
3. 经典场景:获取外部资源

----
## componentWillUnmount

1. 组件移除时被调用
2. 经典场景:资源释放

----
## getSnapshortBeforeUpdate()

1. 在页面 render 之前调用, state 已更新
2. 经典场景:获取 render 之前 的 DOM 状态

----
## componentDidUpdate()

1. 每次 UI 更新时被调用
2. 经典场景:页面需要根据 props 变化重新获取数据

----
## shouldComponentUpdate()

1. 决定 Virtual DOM 是否要重绘
2. **一般可以用 PureComponent 自动实现**
3. 经典场景:性能优化


# 小结
1. 理解 React 组件的生命周期方法
2. 理解生命周期的使用场景
