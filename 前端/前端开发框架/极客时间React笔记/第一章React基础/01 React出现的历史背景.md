#  React 出现的历史背景
![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g0ly95rcktj30a903s407.jpg)
> FaceBook出现图片交互bug

**问题的根源**

1. 传统 UI 操作关注太多细节
2. 应用程序状态分散在各处,难以追踪和维护

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0lybwrc2uj30hi0aognq.jpg)
> 解决:**React始终整体"refresh" page,不需要关注细节**

示例：

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0lydbvlyij30hq0b50uk.jpg)
> 不管消息前后那一条,我直接产生几条消息

React 很简单
1. 1个新概念-**组件描述UI**
2. 4个必须的API
3. 单向数据流-不是双向的
4. 完善的错误提示

React:数据模型问题
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0lyg1v0jmj30g60au0vv.jpg)

> 传统 MVC 结构复杂,多model 和 view,一旦出现错误不能快速发现是 model 错误 还是 View 的错误

Flux架构(设计模式):单向数据流
![](http://ww1.sinaimg.cn/large/006rAlqhly1g0lyinp4j7j30es085ac0.jpg)

它是一个**设计模式：核心思想就是单向数据流；状态管理**

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0lyldm2ntj30ea0990uf.jpg)

# 小结
1. 传统 Web UI 开发问题
2. React:始终整体刷新页面
3. Flux:单向数据流
