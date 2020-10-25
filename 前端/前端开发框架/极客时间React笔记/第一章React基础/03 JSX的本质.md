# JSX 的本质：动态创建组件的语法糖
> 不是模版语言,只是一种**语法糖**

**JSX:在 JavaScript 代码中直接写 HTML 标记**

1. 声明式HTML标记创建组件
```js
const name = 'Nate Wang';
const element = <h1>Hello,{name}</h1>
```

2. **JSX 本质是 js 语法 动态创建组件**

下面元素类同于`document.createElement()`

```js
const name ='Josh Perez';
const element = React.createElement(
    'h1',  //标记的类型
    null,  //属性
    'Hello, ', //children子元素类型
    name    //children 内容
);
```

----

1. 声明 HTML 标记创建组件（JSX）
```js
class CommentBox extends React.Component{
    render(){
        return{
            <div className="comments">
                <h1>Comments ({this.state.items.length})</h1>
                <CommentList data={this.state.items}/>
                <CommentForm />
            </div>
        }
    }
}
```
2. JavaScript 语法动态创建组件

```js
class ComentBox extends React.Component{
    render(){
        return React.createElement(
            "div",  // 标记
            { className:"coments"}, // 属性
            React.createElement( 
                "h1",// 标签
                null,// 属性
                "Comments (",// 组件标签
                this.state.items.length, // 属性
                ")",
            ),
            React.createElement(ComentList.{data:this.state.items}),
            React.createElement{CommentForm,null}
        )
    }
}

ReactDOM,render(
    React.CreateElement(CommentBox,{topicId:"1"}),
    mountNode
)
```

## 在 JSX 中使用表达式
1. JSX 本身也是表达式
```js
const element = <h1>Hello,world</h1>;
```
2. 在属性中使用表达式
```js
<MyComponent foo={1 + 2 + 3 + 4}/>
```
3. 延展属性-给组件传一组值,属性Object-ES6中**延展操作符** ...
```js
const prps = {firstName:'Ben',lastName:'Hector'};
const greeting = <Greeting{...props}/>
```

4. 表达式作为子元素
```js
const element = <li>{props.message}</li>
```

**JSX 优点**

1. 声明式创建界面的直观
2. 代码动态创建界面的灵活
3. 无需学习新的模版语言



**约定:自定义组件必须以大写字母开头**

1. React 认为小写的 tag 是 原生 DOM 节点,如 div
2. **大写字母开头为自定义组件**
3. JSX 标记可以直接使用属性语法, 例如<menu.Item />

# 小结
1. JSX 的本质
2. 如何使用 JSX
3. JSX 优点
