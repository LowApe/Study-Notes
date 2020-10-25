# Readux(2): 深入理解 Store, Action,Reducer

**理解 Store**
```js
const store = createStore(reducer);
```
store 三个方法
1. getState()
2. dispatch(action)
3. subscribe(listener)

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0r3ht6bfyj303h0480sz.jpg)

----
**理解 action**
```js
{
    type:ADD_TOD,
    text: 'Build my first Redux app'
}
```
```js
function todoApp(state = initialState,action){
    switch (action.type) {
        case 'ADD_TODO':
            return Object.assign({},state,{
                todos:[
                    ...state.todos,
                    {
                        text:action.text,
                        completed:false
                    }
                ]
            })
        default:
            return state    
    }
}
```
> reducer 接收所有 action ,上面 action.type 识别 action

![整个数据流向](http://ww1.sinaimg.cn/large/006rAlqhly1g0r3o0y16zj30m90bajvm.jpg)

----

**理解 combineReducers**

```java
export default function todos(state = [],action){
    switch (action.type) {
        case 'ADD_TODO':
            return state.concat([action.text])
        default:
            return state;
    }
}

```

```java
export default function counter(state = 0,action){
    switch (action.type) {
        case 'INCREAMENT':
            return state+1
        case 'DECREMENT':
            return state-1
        default:
            return state;
    }
}
```
如何将两个结合,一个计数,一个todo

```java
import {combineReducers } from 'redux';
import todos from './todos';
import counter form './counter';
export default combineReducers({
    todos,
    counter
})
```

----

**理解 bindActionCreators**

```java
function addTodoWithDispatch(text){
    const aciton={
        type:ADD_TODO,
        text
    }
    dispatch(action)
}
```

```java
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

```java
const boundAddTodo = text => dispatch(addTodo(text))
const boundCompleteTodo = index => dispatch(completeTodo(index))
```
----

```java
import React from "react";
import {
  createStore,
  combineReducers,
  bindActionCreators
} from "redux";

function run() {
  // Store initial state
  const initialState = { count: 0 };

  // reducer
  const counter = (state = initialState, action) => {
    switch (action.type) {
      case "PLUS_ONE":
        return { count: state.count + 1 };
      case "MINUS_ONE":
        return { count: state.count - 1 };
      case "CUSTOM_COUNT":
        return {
          count: state.count + action.payload.count
        };
      default:
        break;
    }
    return state;
  };

  const todos = (state = {}) => state;

  // const store = createStore(counter);
  // Create store
  const store = createStore(
    combineReducers({
      todos,
      counter
    })
  );

  // Action creator
  function plusOne() {
    // action
    return { type: "PLUS_ONE" };
  }

  function minusOne() {
    return { type: "MINUS_ONE" };
  }

  function customCount(count) {
    return { type: "CUSTOM_COUNT", payload: { count } };
  }

  plusOne = bindActionCreators(plusOne, store.dispatch);

  store.subscribe(() => console.log(store.getState()));
  // store.dispatch(plusOne());
  plusOne();
  store.dispatch(minusOne());
  store.dispatch(customCount(5));
}
export default () => (
  <div>
    <button onClick={run}>Run</button>
    <p>* 请打开控制台查看运行结果</p>
  </div>
);

```

![](http://ww1.sinaimg.cn/large/006rAlqhly1g0rnkabx6gj30a506paa7.jpg)

> bindActionCreators(action,dispatch)

# 小结
1. Redux 的基本概念
2. combineReducers
3. bindActionCreators
