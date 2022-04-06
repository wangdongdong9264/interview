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
