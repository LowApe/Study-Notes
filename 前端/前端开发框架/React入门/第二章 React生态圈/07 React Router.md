# 路由不只是页面切换,更是代码组织方式

## React Router
> 为什么需要前端路由
1. 单页应用需要进行页面切换
2. 通过 URL 可以定位到页面
3. 更有语义的组织资源

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0rwabjrgvj30kz0agwgn.jpg)

## React Rounter 的实现
```js
import React from "react";
import {
  // HashRouter as Router,
  BrowserRouter as Router,
  Route,
  Link
} from "react-router-dom";
import { MemoryRouter } from "react-router";

const Home = () => <h1>Home</h1>;
const Hello = () => <h1>Hello</h1>;
const About = () => <h1>About Us</h1>;

export default class RouterSample extends React.PureComponent {
  render() {
    return (
      <Router>
        <div>
          <ul id="menu">
            <li>
              <Link to="/home">Home</Link>
            </li>
            <li>
              <Link to="/hello">Hello</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
          </ul>

          <div id="page-container">
            <Route path="/home" component={Home} />
            <Route path="/hello" component={Hello} />
            <Route path="/about" component={About} />
          </div>
        </div>
      </Router>
    );
  }
}
```
## React Rounter 的特性
1. 声明式路由定义
2. 动态路由

## 三种路由实现方式
1. URL 路径
2. hash 路由-老版本需要刷新#/
3. 内存路由-url不发生变化

## 基于路由配置进行资源组织
1. 实现业务逻辑的松耦合
2. 易于扩展,重构和维护
3. 路由层面实现 Lazy Load

## React Rounter API
1. <Link>：不同链接，不会触发游览器刷新
```js
import { link } form 'react-router-dom';
<Link to="/about">About</Link>
```

2. <NavLink>:类型 Link 但是会添加当前选中状态
```js
<NavLink to="/faq" activeClassName="selected">FAQs</NavLink>
```

3. <Prompt>:满足条件时提示用户是否离开当前页面
```js
import {Prompt } from 'react-router';
<Prompt when={formIsHalfFilledOut} message="Are you sure you want to leave?" />
```

4. <Redirect>:重定向当前页面,例如登录判断
```js
import {Route,Redirect} from 'react-router'
<Route exact path="/" render={()=>(
    loggedIn ?(
        <Redirect to="/dashboard"/>
    ):(
        <PublicHomePage/>
    )
    )}/>
```

5. <Route>: 路径匹配时显示对应组件
```js
import {BrowserRouter as Router,Route} from 'react-rounter-dom';
<Router>
    <div>
        <Route exact path="/" component={Home}/>
        <Route path="/news" component={NewsFeed}>
    </div>
</Router>
```
> exact 是否精确匹配 path

6. <Switch>:只显示第一个匹配的路由
```js
import {Switch,Route} from 'react-router';
<Switch>
    <Route exact path="/" component ={Home}>
    <Route  path="/about" component ={About}>
    <Route  path="/:user" component ={User}>
    <Route   component ={NoMatch}>
</Switch>
```
> 显示 switch 里 第一个 route

# 小结
1. 前端路由是什么
2. React Router 如何实现路由
3. 基于路由思考资源的组织
4. React Router 核心 API
