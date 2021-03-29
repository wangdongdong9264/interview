# vue

## Vue 的响应式原理中 Object.defineProperty 有什么缺陷？为什么在 Vue3.0 采用了 Proxy，抛弃了 Object.defineProperty？

  原因如下：

  1. Object.defineProperty 无法低耗费的监听到数组下标的变化，导致通过数组下标添加元素，不能实时响应；
  2. Object.defineProperty 只能劫持对象的属性，从而需要对每个对象，每个属性进行遍历。如果属性值是对象，还需要深度遍历。 Proxy 可以劫持整个对象， 并返回一个新的对象
  3. Proxy 不仅可以代理对象，还可以代理数组。还可以代理动态增加的属性

## vue针对数组如何实现双向绑定

Object.definePropert只能把对象属性改为getter/setter，而对于数组的方法就无能为力了，其内部巧妙的使用了数组的属性来实现了数据的双向绑定

```js

let obarr = [] // 需要监听的数组
const arrayProto = Array.prototype 
const arrayMethods = Object.create(arrayProto) // copy一份数组的原型方法， 防止污染原生方法
Object.defineProperty(arrayMethods,'push',{
    value:function mutator(){
        console.log('obarr.push会走这里')
    }
}) //
obarr.__proto__ = arrayMethods; // 使用arrayMethods覆盖obarr的所有方法

```

他的方法同理，我们只需要把所有需要实现的方法循环遍历执行即可

## 组件中的data为什么是函数

为什么组件中的data必须是一个函数，然后return一个对象，而new Vue实例里，data可以直接是一个对象？

因为组件是用来复用的，JS里对象是引用关系，这样作用域没有隔离，而new Vue的实例，是不会被复用的，因此不存在引用对象问题

## v-for和v-if为什么不要一起使用，优先级如何

当v-if与v-for一起使用时，v-for具有比v-if更高的优先级，这意味着 v-if 将分别重复运行于每个 v-for 循环中
所以，不推荐v-if和v-for同时使用

vue3.x版本中修改了优先级  v-if的优先级大于v-for

## 写 React / Vue 项目时为什么要在列表组件中写 key，其作用是什么

  vue 和 react 都是采用 diff 算法来对比新旧虚拟节点，从而更新节点。在 vue 的 diff 函数交叉对比中，当新节点跟旧节点头尾交叉对比没有结果时，会根据新节点的 key 去对比旧节点数组中的 key，从而找到相应旧节点（这里对应的是一个 key => index 的 map 映射）。如果没有找到就认为是一个新增节点。而如果没有 key，那么就会采用遍历查找的方式去找到对应的旧节点。一种一个 map 映射，另一种是遍历查找。相比而言，map 映射的速度更快。

## 在 Vue 中，子组件为何不可以修改父组件传递的 Prop，如果修改了，Vue 是如何监控到属性的修改并给出警告的

  1. 因为 Vue 是单项数据流，易于检测数据的流动，出现了错误可以更加迅速的定位到错误发生的位置；
  2. 通过 setter 属性进行检测，修改值将会触发 setter，从而触发警告

## provide/inject原理

依赖注入，其核心原理就是通过$parent向上查找祖先组件中的provide，找到则赋值给对应的inject即可

`inject、provide`的初始化时间在生命周期钩子函数beforeCreate之后，created之前

`initInjections(vm)` 解析inject是在初始化data/props之前

`resolveInject`函数，功能是通过$parent一层层向上查找祖先节点的数据，直到找到对应于inject的provide数据

`initProvide(vm)` 解析provide是在初始化data/props之后

这也符合数据初始化的一个处理逻辑

## 双向绑定和 vuex 是否冲突

  当在严格模式中使用 Vuex 时，在属于 Vuex 的 state 上使用 v-model 会导致出错

  解决方案：

  1. 给 `<Input>` 中绑定 value，然后侦听 input 或者 change 事件，在事件回调中调用一个方法
  2. 使用带有 setter 的双向绑定计算属性

## Vue 的父组件和子组件生命周期钩子执行顺序是什么

  1. 加载渲染过程：父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted
  2. 子组件更新过程：父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated
  3. 父组件更新过程：父 beforeUpdate -> 父 updated
  4. 父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed

## vue-router 里的link标签和a标签有什么区别

1. 有 onClick 则执行 OnClick；
2. 阻止 a 标签默认事件（跳转页面）；
3. 在取得跳转 href（to 属性值），用 history/hash 跳转，此时只是链接发现改变，并没有刷新页面；

## vue组件的几种写法

1. Vue.extend
2. Vue.component
3. render

## new Vue 以后发生的事情

1. new Vue 会调用 Vue 原型链上的 _init 方法对 Vue 实例进行初始化；
2. 首先是 initLifecycle 初始化生命周期，对 Vue 实例内部的一些属性（如 children、parent、isMounted）进行初始化；
3. initEvents，初始化当前实例上的一些自定义事件（Vue.$on）；
4. initRender，解析 slots 绑定在 Vue 实例上，绑定 createElement 方法在实例上；
5. 完成对生命周期、自定义事件等一系列属性的初始化后，触发生命周期钩子 beforeCreate；
6. initInjections，在初始化 data 和 props 之前完成依赖注入（类似于 React.Context）；
7. initState，完成对 data 和 props 的初始化，同时对属性完成数据劫持内部，启用监听者对数据进行监听（更改）；
8. initProvide，对依赖注入进行解析；
9. 完成对数据（state 状态）的初始化后，触发生命周期钩子 created；
10. 进入挂载阶段，将 vue 模板语法通过 vue-loader 解析成虚拟 DOM 树，虚拟 DOM 树与数据完成双向绑定，触发生命周期钩子 beforeMount；
11. 将解析好的虚拟 DOM 树通过 vue 渲染成真实 DOM，触发生命周期钩子 mounted；

## vue 架构

mvvm
