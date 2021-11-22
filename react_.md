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
