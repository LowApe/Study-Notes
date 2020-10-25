# 理解 Virtual DOM 及 key 属性的作用

JSX 的运行基础: Virtual DOM
`Diff 算法 `区别 tag 不同

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0m840ojf3j30kx0d5q51.jpg)


![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0m85i8wgaj30ly0de0v2.jpg)

----

1. 根节点比较
2. 下一层发现,属性变化及顺序
3. 节点类型发生变化-> 删除旧节点然后创建

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0m89h0lggj30lh0a7ta2.jpg)

----
节点跨层移动
1. B 子节点没有就直接删除
2. 搜索下层,发现 D节点直接重新创建


![](http://ww1.sinaimg.cn/large/006rAlqhly1g0m8adze9pj30lb09n0ud.jpg)

----

虚拟 DOM 的两个假设
1. 组件的 DOM 结构是相对稳定的
2. 类型相同的兄弟节点可以被唯一标识 key

# 小结
1. 算法复杂度为 O(n)
2. 虚拟 DOM 如何计算 diff
3. key 属性的作用
