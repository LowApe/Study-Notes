# 理解 Context API 及其使用场景
React 16.3 新特性:Context API
解决组件之间通信问题

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0n8jehuvij30ix08b405.jpg)
1. 根节点提供所有上下文数据
2. 其他节点通过 Context API 访问数据

```js
//创建一个上下文
const ThemeContext = React.createContext('light');

class App extends React.Component{
    render(){
        return (
            <ThemeContext.Provider value="dark">
                <ThemeButton />
            </ThemeContext.Provider>
        );
    }
}

function ThemeButton(props){
    return (
        <ThemeContext.Consumer>
            {theme =><Button {...props}theme={theme} />}
        </ThemeContext.Consumer>
    )
}
```
> provider value 是多少,下面消耗的值就是多少


# 小结
1. Context API 的使用方法
2. 使用场景
