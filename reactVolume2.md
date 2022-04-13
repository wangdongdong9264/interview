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

1. 在组件之间复用状态逻辑很难

可以使用hook从组件中提取状态逻辑，使得这些逻辑可以单独测试并复用。hook使我们在无需修改组件结构的情况下复用状态逻辑。这使得在组件间或社区共享hook变得更加便捷

2. 复杂组件变得难以理解

在组件中，每个生命周期常常包含一些不相关的逻辑，组件常常在 componentDidMount 和 componentDidUpdate 中获取数据。但是，同一个 componentDidMount 中可能也包含很多其它的逻辑，如设置事件监听，而之后需在 componentWillUnmount 中清除。相互关联且需要对照修改的代码被进行了拆分，而完全不相关的代码却在同一个方法中组合在一起。如此很容易产生 bug，并且导致逻辑不一致

hook将组件中相互关联的部分拆成更小的函数，而非强制按照生命周期划分，还可以使用reducer 来管理组件内部的状态

3. 难以理解的class

必须理解js 中this的工作方式，还不能忘记绑定事件

hook使你在非class的情况下可以使用更多的react 特性。无需学习复杂的函数式或响应式编程

### React Hook 的使用限制有哪些

1. 不要在循环，条件或嵌套函数中使用hook
2. 在react的函数组件中调用hook

为什么不要在循环、条件或嵌套函数中调用hook呢

因为hooks的设计是基于数组实现。在调用时按顺序加入数组中，如果使用循环、条件或嵌套函数很有可能导致数组取值错位，执行错误的hooks。 实际上react的源码里不是数组，是链表

### useEffect 与 useLayoutEffect 的区别

共同点

* 运用效果： useEffect 与 useLayoutEffect 两者都是用于处理副作用，这些副作用包括改变dom，定时器等操作。在函数组件内部操作副作用是不被允许对的，所以需要使用这两个函数去处理
* 使用方式： useEffect 与 useLayoutEffect 两者底层的函数签名是完全一致的，都是调用的mountEffectImpl方法，在使用上也没有什么差异，基本可以直接替换

不同点

* 使用场景： useEffect在react的渲染过程中是被异步调用的，用于绝大多数场景；useLayoutEffect会在所有的dom变更之后在同步调用，主要用于处理dom操作，调整样式，避免页面闪烁等。所以需要避免做一些计算量较大的耗时任务，从而导致堵塞
* 使用效果： useEffect是按照顺序执行代码的， 改变屏幕像素之后执行（先渲染，后改变dom），容易产生闪烁现象；useLayoutEffect是改变屏幕像素之前就执行了（会推迟页面显示的事件，先改变dom，后渲染），不会产生闪烁。useLayoutEffect总是比useEffect先执行

### React Hooks在平时开发中需要注意的问题和原因

* 不要在循环，条件或嵌套函数中调用Hook，必须始终在 React函数的顶层使用Hook

因为React需要利用调用顺序来正确更新相应的状态，以及调用相应的钩子函数。一旦在循环或条件分支语句中调用Hook，就容易导致调用顺序的不一致性

* 使用useState时候，使用push，pop，splice等直接更改数组对象的坑

使用push直接更改数组无法获取到新值，应该采用析构方式。在class里不会有这种情况

```js

import {useState} from "react";
function Indicatorfilter() {
  let [num, setNums] = useState([0, 1, 2, 3])
  const test = () => {
    // 这里坑是直接采用push去更新num
    // setNums(num)是无法更新num的
    // 必须使用num = [...num ,1]
    num.push(1)
    // num = [...num ,1]
    setNums(num)
  }
  return (
    <div className='filter'>
      <div onClick={test}>测试</div>
      <div>
        {num.map((item, index) => (
          <div key={index}>{item}</div>
        ))}
      </div>
    </div>
  )
}

export default Indicatorfilter

```

* useState设置状态的时候，只有一次生效，后期需要更新状态必须通过useEffect

TableDeail是一个公共组件，在调用它的父组件中，改变了columns的值，tabColumn的值 不随者columns更新而更新

```js

const TableDeail = ({
    columns,
}:TableData) => {
    const [tabColumn, setTabColumn] = useState(columns) 
}

// 正确的做法是通过useEffect改变这个值
const TableDeail = ({
    columns,
}:TableData) => {
    const [tabColumn, setTabColumn] = useState(columns) 
    useEffect(() =>{setTabColumn(columns)},[columns])
}

```

* 善用useCallback

父组件传递给组件事件句柄时，如果我们没有任何参数变动可能会选用useMemo，但是每一次父组件渲染子组件即使没有变化也会跟者渲染一次

* 不要滥用useContext

可以使用 redux 等状态管理工具

### React Hooks 和生命周期的关系
