# 理解异步 Action,Redux 中间件
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0rp3ia7dnj30lc0ayq7h.jpg)

**Redux 中间件 (Middleware)**

Middleware 截获某种特种类型 action
ajax 并不是标志 action ,而是 action 设计模式`异步 action 是几个同步 action 组成`

1. 激活 action
2. 发出 action


## 小结
1. 异步 action 不是特殊 action<br>
而是多个同步 action 的组合使用时
2. 中间件在 dispatcher 中截获 action 做特殊处理
