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

| class 组件      | Hooks 组件 |
| constructor      | useState |
| getDerivedStateFromProps      | useState 里面 update 函数 |
| shouldComponentUpdate      | useMemo |
| render      | 函数本身 |
| componentDidMount      | useEffect |
| componentDidUpdate      | useEffect |
| componentWillUnmount      | useEffect 里面返回的函数 |
| componentDidCatch      | 无 |
| getDerivedStateFromError     | 无 |

## 虚拟dom

### 对虚拟 DOM 的理解？虚拟 DOM 主要做了什么？虚拟 DOM 本身是什么？

本质上来说，virtual Dom是一个js对象，通过对象的方式来表达dom结构。将页面的状态抽象位js对象的形势，配合不同的渲染工具，使跨平台渲染成为可能。通过事务处理机制，将多次dom修改的结果一次性的更新到页面上，从而有效的减少页面渲染的次数，减少修改dom的重绘重排次数，提高渲染性能。

虚拟dom是对dom的抽象，这个对象是更加轻量的对dom的描述，它设计最初目的，就是更好的跨平台，比如nodejs中没有dom，如果想实现ssr，那么就是借助虚拟dom，因为虚拟dom本身是js对象。在代码渲染到页面之前，vue/react会把代码转换成一个对象（虚拟dom）。以对象的形式来描述真实的dom结构，最终渲染到页面。在每次数据发生变化前，虚拟dom都会缓存一份，变化时，当前虚拟dom会与缓存的虚拟dom进行比较。在vue或者react内部封装了diff算法来比较变化，最后渲染变化的

### 为什么要用 Virtual DOM

1. 保证了性能下限，在不进行手动优化的情况下，提供过的去的性能
2. 跨平台

对比真实dom操作和 VDom操作

* 真实dom：生成字符串模版 + 重建所有dom元素
* VDom： 生成vNode + DOMDiff + 必要的dom更新

Vdom的更新dom的准备工作消耗更多的时间，相比于更多dom操作消费是及其便宜的。

Vdom本质上是js的对象，它可以很方便的跨平台操作，ssr，HybridApp

### React diff 算法的原理是什么

1. 真实的dom会映射为虚拟dom
2. 虚拟dom变化后，计算差异生成patch
3. 根据patch去更新真实的dom

diff算法可以总结为三个策略，分别从树，组件以及元素

* 策略1：忽略节点夸层级操作场景 基于树进行比较

    即对树进行分层比较。即同一层次的节点进行比较，如果发现节点已经不存在了，则该节点及子节点会被完全删除掉，不会进行深入比较

* 策略2: 如果如果组件的class一致，则默认为相识的树结构，否则默认为不同的树结构。基于组件对比

    在组件对比的过程中，如果组件是同一类型则进行树比较，不是就放入布丁中。这也就是为什么 shouldComponentUpdate、PureComponent 及 React.memo 可以提高性能的原因

* 策略3: 同一层级的子节点，可以通过标记key的方式进行列表对比。（基于节点进行对比）

    在diff算法中react会借助元素的key值来判断该元素是新创建还是被移动的，从而减少不必要的渲染

    react还需要借助key值来判断元素与本地状态的关联关系

### 虚拟 DOM 的引入与直接操作原生 DOM 相比，哪一个效率更高，为什么

虚拟dom相对原生的dom操作不一定效率更高，如果只是修改一个按钮的文案，那么虚拟dom的操作无论如何都比不过真实的dom操作。

在首次渲染大量dom时，由于多了一层虚拟dom的计算，所以也比不上真实dom

他能保证性能下限，在真实dom操作的时候进行针对性的优化，具体块慢需要根据具体场景来说

在整个dom操作的演化过程中，其实主要矛盾并不在于性能，而在于开发体验/效率，虚拟dom正是前端为了最求更好的研发体验和研发效率而创造出来的产物。

虚拟dom的优越之处在于，它能够在提供，友好，高效的研发模式的同时，仍然保持一个不错的性能

### React 与 Vue 的 diff 算法有何不同

diff算法是指生成更新补丁的方式，主要应用于虚拟dom树变化后，更新真实dom，所以diff算法一定存在这样一个过程：
触发更新-》生成补丁-》应用补丁

react的diff算法，触发更新的时机主要在state变化与hook调用之后。此时触发了dom树变更遍历，采用了深度优先遍历算法。

为了优化效率，使用了分治法的方式。将单一节点转换了3种类型节点的对比，

* 树比对：由于网页视图中较少有跨层级节点移动，两株虚拟 DOM 树只对同一层次的节点进行比较
* 组件比对：如果组件是同一类型，则进行树比对，如果不是，则直接放入到补丁中
* 元素比对：主要发生在同层级中，通过标记节点操作生成补丁，节点操作对应真实的 DOM 剪裁操作

react 16之后，引入Fiber架构，为了使整个更新过程可以随时暂停恢复，节点与树分别采用了FiberNode 与 FiberTree 进行重构。

fiberNode 使用了双链表的结构，可以直接找到兄弟节点与子节点。整个更新过程由 current 与 workInProgress 两株树双缓冲完成。workInProgress 更新完成后，再通过修改 current 相关指针指向新节点。、

Vue的整体diff策略与react对其，虽然缺乏时间切片能力，但这并不意味着vue的性能更差，因为vue 3初期也引用过，后期因为收益不高移除掉了。除了高帧动画，在vue中其它场景几乎都可以使用防抖和截留提高性能

## 其它
