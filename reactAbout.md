# react 周边

## 对 Redux 的理解，主要解决什么问题

redux 提供了一个叫store的统一仓储库，组件通过`dispatch`将`state`直接传入`store`，不用通过其它的组件，并且组件通过`subscribe`从`store`获取到state的改变。使用了redux，所有的组件都可以从`store`中获取到所需要的`state`，他们也能从`store`获取到`state`的改变。这比组建之间相互传递数据清晰的多

主要解决的问题：单纯的`redux`只是一个状态机，是没有ui呈现的，`react-redux`作用是将`redux`的状态机和react的ui呈现绑定在一起，当你`dispatch action`改变`state`的时候，会自动更新页面

## redux 原理及工作流程

原理 Redux源码主要分为以下几个模块文件

* compose.js 提供从右到左进行函数式编程
* createStore.js 提供作为生成唯一store的函数
* combineReducers.js 提供合并多个reducer的函数，保证store的唯一性
* bindActionCreators.js 可以让开发者在不直接接触dispacth的前提下进行更改state的操作
* applyMiddleware.js 这个方法通过中间件来增强dispatch的功能

---

工作流程

* `const store = createStore(fn)`生成数据;
* `action: {type: Symble('action01), payload:'payload' }`定义行为;
* dispatch发起`action：store.dispatch(doSomething('action001'))`
* reducer：处理action，返回新的state;

通俗点解释:

* 首先，用户（通过View）发出Action，发出方式就用到了dispatch方法
* 然后，Store自动调用Reducer，并且传入两个参数：当前State和收到的Action，Reducer会返回新的State
* State—旦有变化，Store就会调用监听函数，来更新View

以 store 为核心，可以把它看成数据存储中心，但是他要更改数据的时候不能直接修改，数据修改更新的角色由Reducers来担任，store只做存储，中间人，当Reducers的更新完成以后会通过store的订阅来通知react component，组件把新的状态重新获取渲染，组件中也能主动发送action，创建action后这个动作是不会执行的，所以要dispatch这个action，让store通过reducers去做更新React Component 就是react的每个组件

## Redux 中异步的请求怎么处理

可以在 `componentDidmount` 中直接进⾏请求⽆须借助`redux`。但是在⼀定规模的项⽬中,上述⽅法很难进⾏异步流的管理,通常情况下我们会借助`redux`的异步中间件进⾏异步处理。`redux`异步流中间件其实有很多，当下主流的异步中间件有两种`redux-thunk`、`redux-saga`。

使用react-thunk中间件：

redux-thunk优点:

* 体积⼩: redux-thunk的实现⽅式很简单,只有不到20⾏代码
* 使⽤简单: redux-thunk没有引⼊像redux-saga或者redux-observable额外的范式,上⼿简单

redux-thunk缺陷:

* 样板代码过多: 与redux本身⼀样,通常⼀个请求需要⼤量的代码,⽽且很多都是重复性质的
* 耦合严重: 异步操作与redux的action偶合在⼀起,不⽅便管理
* 功能孱弱: 有⼀些实际开发中常⽤的功能需要⾃⼰进⾏封装

---

使用redux-saga中间件:

redux-saga优点:

* 异步解耦: 异步操作被被转移到单独 `saga.js` 中，不再是掺杂在 `action.js` 或 `component.js` 中
* action摆脱thunk function: dispatch 的参数依然是⼀个纯粹的 action (FSA)，⽽不是充满 “⿊魔法” thunk function
* 异常处理: 受益于 generator function 的 saga 实现，代码异常/请求失败 都可以直接通过 try/catch 语法直接捕获处理
* 功能强⼤: redux-saga提供了⼤量的Saga 辅助函数和Effect 创建器供开发者使⽤,开发者⽆须封装或者简单封装即可使⽤
* 灵活: redux-saga可以将多个Saga可以串⾏/并⾏组合起来,形成⼀个⾮常实⽤的异步flow
* 易测试，提供了各种case的测试⽅案，包括mock task，分⽀覆盖等等

redux-saga缺陷:

* 额外的学习成本: redux-saga不仅在使⽤难以理解的 generator function,⽽且有数⼗个API,学习成本远超redux-thunk,最重要的是你的额外学习成本是只服务于这个库的
* 体积庞⼤: 体积略⼤,代码近2000⾏，min版25KB左右
* 功能过剩: 实际上并发控制等功能很难⽤到,但是我们依然需要引⼊这些代码
* ts⽀持不友好: yield⽆法返回TS类型

## Redux 怎么实现属性传递，介绍下原理

react-redux 数据传输∶ `view` ==> `action` ==> `reducer` ==> `store` ==> `view`

* view 上的AddClick 事件通过mapDispatchToProps 把数据传到action ---> `click:()=>dispatch(ADD)`
* action 的ADD 传到`reducer`上
* reducer传到store上 `const store = createStore(reducer)`
* store再通过 mapStateToProps 映射穿到view上`text:State.text`

## Redux 中间件是什么？接受几个参数？柯里化函数两端的参数具体是什么？

Redux 的中间件提供的是位于 action 被发起之后，到达 reducer 之前的扩展点，换而言之，

原本 `view` => `action` => `reducer` => `store` 的数据流加上中间件后

变成了 `view` -> `action` -> `middleware` -> `reducer` -> `store` ，

在这一环节可以做一些"副作用"的操作，如异步请求、打印日志等

---

```js

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

从applyMiddleware中可以看出∶

* redux中间件接受一个对象作为参数，对象的参数上有两个字段 dispatch 和 getState，分别代表着 Redux Store 上的两个同名函数。
* 柯里化函数两端一个是 middewares，一个是store.dispatch

## Redux 请求中间件如何处理并发


