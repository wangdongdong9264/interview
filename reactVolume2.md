# react

## redux

### 对 Redux 的理解，主要解决什么问题

React是视图层框架。

Redux是一个用来管理数据状态和UI状态的JavaScript应用工具。

随着JavaScript单页应用（SPA）开发日趋复杂， JavaScript需要管理比任何时候都要多的state（状态）， Redux就是降低管理难度的

Redux 提供了一个叫 store 的统一仓储库，
组件通过 dispatch 将 state 直接传入store，不用通过其他的组件。
并且组件通过 subscribe 从 store获取到 state 的改变。
使用了 Redux，所有的组件都可以从 store 中获取到所需的 state，他们也能从store 获取到 state 的改变。
这比组件之间互相传递数据清晰明朗的多

主要解决的问题： 单纯的Redux只是一个状态机，是没有UI呈现的，react- redux作用是将Redux的状态机和React的UI呈现绑定在一起，当你dispatch action改变state的时候，会自动更新页面

### Redux 原理及工作流程

原理 Redux源码主要分为以下几个模块文件:

* `compose.js` 提供从右到左进行函数式编程
* `createStore.js` 提供作为生成唯一store的函数
* `combineReducers.js` 提供合并多个reducer的函数，保证store的唯一性
* `bindActionCreators.js` 可以让开发者在不直接接触dispacth的前提下进行更改state的操作
* `applyMiddleware.js` 这个方法通过中间件来增强dispatch的功能

工作流程:

* `const store = createStore(fn)` 生成数据;
* `action: {type: Symble('action01), payload:'payload' }` 定义行为;
* dispatch发起action `store.dispatch(doSomething('action001'))`;
* reducer `处理action，返回新的state`

### Redux 怎么实现属性传递，介绍下原理

react-redux 数据传输：View --》action --》render --》store --》View

* View上的addClick事件通过mapDispatchToProps把数据传递到 action --》`click:()=>dispatch(ADD)`
* action的ADD传到 reducer
* reducer传到store上 `const store = createStore(reducer)`
* store再通过 mapStateToProps 映射穿到view上text:State.text

```jsx

import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import { Provider, connect } from 'react-redux';
class App extends React.Component{
    render(){
        let { text, click, clickR } = this.props;
        return(
            <div>
                <div>数据:已有人{text}</div>
                <div onClick={click}>加人</div>
                <div onClick={clickR}>减人</div>
            </div>
        )
    }
}
const initialState = {
    text:5
}
const reducer = function(state,action){
    switch(action.type){
        case 'ADD':
            return {text:state.text+1}
        case 'REMOVE':
            return {text:state.text-1}
        default:
            return initialState;
    }
}

let ADD = {
    type:'ADD'
}
let Remove = {
    type:'REMOVE'
}

const store = createStore(reducer);

let mapStateToProps = function (state){
    return{
        text:state.text
    }
}

let mapDispatchToProps = function(dispatch){
    return{
        click:()=>dispatch(ADD),
        clickR:()=>dispatch(Remove)
    }
}

const App1 = connect(mapStateToProps,mapDispatchToProps)(App);

ReactDOM.render(
    <Provider store = {store}>
        <App1></App1>
    </Provider>,document.getElementById('root')
)

```

### Redux 中间件是什么？接受几个参数？柯里化函数两端的参数具体是什么？

redux的中间件提供的是位于action被发起之后，到达reducer之前的扩展点，换而言之，原本 view --》 action --》reducer --》 store的数据流 加上中间件后变成了 view --》 view --》 action --》 middleware --》 reducer --》 store 在这一环节可以做一些“副作用”的操作，如异步请求，打印日志等

```js

// applyMiddleware源码

export default function applyMiddleware(...middlewares) {
    return createStore => (...args) => {
        // 利用传入的createStore和reducer和创建一个store
        const store = createStore(...args)
        let dispatch = () => {
            throw new Error()
        }
        const middlewareAPI = {
            getState: store.getState,
            dispatch: (...args) => dispatch(...args)
        }
        // 让每个 middleware 带着 middlewareAPI 这个参数分别执行一遍
        const chain = middlewares.map(middleware => middleware(middlewareAPI))
        // 接着 compose 将 chain 中的所有匿名函数，组装成一个新的函数，即新的 dispatch
        dispatch = compose(...chain)(store.dispatch)
        return {
            ...store,
            dispatch
        }
    }
}

```

从 applyMiddleware 函数中可以看出

* redux中间件接收一个对象作为参数，对象的参数上有两个字的 dispatch 和 getState, 分别代表着Redux store上的两个同名参数
* 柯里化函数两端一个是 middewares，一个是store.dispatch

### Redux 请求中间件如何处理并发

使用redux-Sage 是一个管理redux应用异步操作的中间件

通过创建Sagas 将所有的异步操作逻辑放在一个地方进行集中处理，以此将react中的同步操作与异步操作区分开来，以便后期的管理与维护

```js

// reddux-Sager如何处理并发
// takeEvery 可以让多个sage任务并行被fork执行

import {
    fork,
    take
} from "redux-saga/effects"

const takeEvery = (pattern, saga, ...args) => fork(function*() {
    while (true) {
        const action = yield take(pattern)
        yield fork(saga, ...args.concat(action))
    }
})


// takeLatest 不允许多个asga 任务并行 一旦接受到新的action，它就会取消前面所有的fork过的任务

import {
    cancel,
    fork,
    take
} from "redux-saga/effects"

const takeLatest = (pattern, saga, ...args) => fork(function*() {
    let lastTask
    while (true) {
        const action = yield take(pattern)
        if (lastTask) {
            yield cancel(lastTask) // 如果任务已经结束，则 cancel 为空操作
        }
        lastTask = yield fork(saga, ...args.concat(action))
    }
})

```

## Hooks

### 为什么 useState 要使用数组而不是对象

es6的解构赋值

```js

const foo = [1, 2, 3];
const [one, two, three] = foo;
console.log(one); // 1
console.log(two); // 2
console.log(three); // 3

```

* 如果 useState 返回的是数组，那么使用者可以对数组中的元素进行命名，代码看起来也比较干净
* 如果 useState 返回的是对象，在解构对象的时候必须要和 useState 内部返回的对象同名，想要多次的话，必须的设置别名才使用返回值

```js

// 第一次使用
const { state, setState } = useState(false);
// 第二次使用
const { state: counter, setState: setCounter } = useState(0) 

```

总结：useState 返回的是数组而不是对象的原因就是为了降低使用的复杂度

### React Hooks 解决了哪些问题？
