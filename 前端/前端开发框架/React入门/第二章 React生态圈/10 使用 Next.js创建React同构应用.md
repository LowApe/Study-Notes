# 使用 Next.js 创建 React 同构应用

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0rzsevr2xj30gr0cgmyg.jpg)

> 第一次请求服务器渲染,然后直接返回,速度较快

## 创建页面
1. 页面就是 pages 目录下的一个组件
2. static 目录映射静态文件
3. page 具有特殊静态方法 getInitialProps

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0rzvbtzvgj304x06ot99.jpg)

## 在页面中使用其他 React 组件
1. 页面也是标准的 node 模块,可使用其它 React 组件
2. 页面针对性打包,仅包含其引入的组件

## 使用 Link 实现同构路由
1. 使用 "next/link" 定义连接
2. 点击连接时页面不会刷新
3. 使用 prefetch 预加载目标资源(标签,预加载未点击的组件)
4. 使用 replace 属性替换 URL

```js
import Link from 'next/link';
export default () =>{
    <div>
        Click{' '}
        <Link href="/about" prefetch>
            <a>here</a>
        </Link>{' '}
        to read more
    </div>
}

```

## 动态加载页面
```js
import dynamic from 'next/dynamic'
const DynamicComponentWithCustomLoading = dynamic(
    import('../components/hello2'),
    {
        loading:()=><p>...</p>
    }
)

export default ()=>{
    <div>
        <Header />
        <DynamicComponentWithCustomLoading />
        <p>HOME PAGE is here!</p>
    </div>
}
```

# 小结
1. 同构应用概念
2. next.js 基本用法
