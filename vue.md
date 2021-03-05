# vue

## Vue 的响应式原理中 Object.defineProperty 有什么缺陷？为什么在 Vue3.0 采用了 Proxy，抛弃了 Object.defineProperty？

  原因如下：

  1. Object.defineProperty 无法低耗费的监听到数组下标的变化，导致通过数组下标添加元素，不能实时响应；
  2. Object.defineProperty 只能劫持对象的属性，从而需要对每个对象，每个属性进行遍历。如果属性值是对象，还需要深度遍历。 Proxy 可以劫持整个对象， 并返回一个新的对象
  3. Proxy 不仅可以代理对象，还可以代理数组。还可以代理动态增加的属性

## 写 React / Vue 项目时为什么要在列表组件中写 key，其作用是什么

  vue 和 react 都是采用 diff 算法来对比新旧虚拟节点，从而更新节点。在 vue 的 diff 函数交叉对比中，当新节点跟旧节点头尾交叉对比没有结果时，会根据新节点的 key 去对比旧节点数组中的 key，从而找到相应旧节点（这里对应的是一个 key => index 的 map 映射）。如果没有找到就认为是一个新增节点。而如果没有 key，那么就会采用遍历查找的方式去找到对应的旧节点。一种一个 map 映射，另一种是遍历查找。相比而言，map 映射的速度更快。

## 在 Vue 中，子组件为何不可以修改父组件传递的 Prop，如果修改了，Vue 是如何监控到属性的修改并给出警告的

  1. 因为 Vue 是单项数据流，易于检测数据的流动，出现了错误可以更加迅速的定位到错误发生的位置；
  2. 通过 setter 属性进行检测，修改值将会触发 setter，从而触发警告

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
