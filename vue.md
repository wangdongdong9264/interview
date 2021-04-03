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

MVVM

  分为`Model`, `View`, `ViewModel`三种

  1. `Model`代表数据模型，数据和业务逻辑都在这层定义
  2. `View`代表UI视图，负责数据展示
  3. `ViewModel`负责监听`Model`中数据的改变并且控制视图的更新，处理用户操作
  
  Model和View并无直接关联，而是通过ViewModel来进行关联的，Model和ViewModel之间有着双向数据绑定的联系。
  因此当Model中的数据改变时会触发View层的刷新，View中由于用户交互操作而改变的数据也会在Model层同步。
  这种模式实现了Model和view的数据同步, 因此开发者只需要关注对数据的维护操作，不需要自己操作dom

## Computed和Watch的区别

Computed

  1. 它支持缓存，只有依赖的数据发生了变化，才会重新计算
  2. 不支持异步， 当Computed中异步操作时，无法监听数据的变化
  3. Computed的值默认会走缓存, 计算属性是基于它们的响应式依赖进行缓存的，也就是基于data声明过，或者父组件传递过来的Props中的数据
  4. 如果一个属性是由其它属性计算而来的， 这个属性依赖其它的属性，一般会使用computed
  5. 如果Computed属性的属性值是个函数， 那么默认使用get方法，函数的返回值就是属性的属性值；在Computed中属性有一个get方法和一个set方法， 当数据发生变化时，会调用set方法

Watch

  1. 不支持缓存, 数据发生变化时，它就会触发相应操作
  2. 支持异步监听
  3. 监听的函数接收2个参数，第一个是最新值，第二个是变化前的值
  4. 当一个属性发生变化时，就需要执行相应操作
  5. 监听数据必须是data中声明的或者父组件传递过来的props中的数据，当发生变化时，会触发其它操作
     1. immediate：组件加载立即触发回调函数
     2. deep： 深度监听， 发现内部数据变化，在复杂数据类型中使用，例如数组或对象发生变化。需要注意的是，deep无法监听到数组和对象内部的变化

总结

* `Computed`计算属性： 依赖其它属性值，并且computed的值有缓存，只有它依赖的属性值发生改变，下一次获取Computed的值时才会重新计算
* `Watch`监听器：更多的是观察的作用，无缓存性，类似于某些数据的监听回调，每当监听的数据变化时都会执行回调函数
  
使用场景

* 当我们需要进行数值计算，并且依赖于其它数据时，应该使用computed，因为可以利用computed的缓存特性，避免每次获取值时，都要计算
* 当我们需要在数据变化时执行异步或开销比较大的操作时，应该使用watch，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的

## Computed和Methods的区别

  我们可以将同一个函数定义为一个method或者一个计算属性。对于最终结果，两种方式是相同的

  不同点：

  1. `computed`计算属性是基于它们的依赖进行缓存的，只有它的相关依赖发生改变时才会重新计算。对于`methods`而言 只要发生了重新渲染就会调用
  2. methods调用总会执行该函数

## slot是什么，有什么用， 原理是什么

  `slot`又名插槽， 是vue的内容分发机制，组件内部的模版引擎使用slot元素作为承载分发内容的出口。

  插槽slot是子组件的一个模版标签元素，而这一个标签元素是否显示，以及怎么显示是由父组件决定的。

  slot分为三类，默认插槽，具名插槽，作用域插槽

  1. 默认插槽：又名匿名插槽，当slot没有指定name属性的时候一个默认显示插槽，一个组件只有一个匿名插槽
  2. 具名插槽：带有具体名字的插槽，也就是再有name属性的slot，一个组件可以出现多个具名插槽
  3. 作用域插槽：默认插槽，具名插槽的一个变体，可以是匿名插槽，也可以是具名插槽，该插槽的不同点是在子组件渲染作用域插槽时，可以将子组件内部数据传递给父组件，让父组件根据子组件的传递过来的数组决定如何渲染该插槽

实现原理：

  当子组件vm实例化时，获取到父组件传入的slot标签内容，存放在`vm.$slot`中，默认插槽为`vm.$slot.default`, 具名插槽为`vm.$slot.xxx` xxx为插槽名字，当组件执行渲染函数的时候，遇到slot标签，使用`$slot`中的内容进行替换，此时可以为插槽传递数据，若存在数据，则可称该插槽为作用域插槽

## filters过滤器的作用，如何实现一个过滤器

过滤器是用来过滤数据的，在Vue中使用`filters`来过滤数据，过滤器不会修改数据，而是过滤数据，改变用户看到的输出（计算属性`computed`, 方法`methods` 都是会修改数据来处理数据格式的输出显示）

过滤器是一个函数，它会把表达式中的值始终当作函数的第一个参数。过滤器用在插值表达式`{{}}` 和 `v-bind`表达式中，然后放在操作符`|` 后面进行过滤

```js

// 例如，在显示金额，给商品价格添加单位
// <li>商品价格：{{item.price | filterPrice}}</li>

 filters: {
    filterPrice (price) {
      return price ? ('￥' + price) : '--'
    }
  }

```

过滤器串联

  `{{message | filterA | filterB}}` 这个例子中，`filterA`被定义为接收单个参数的过滤器函数，表达式 `message`的值将作为参数传入到函数中，然后继续调用同样被定义为接收单个函数的过滤器`filterB`, 将`filterA`的结果传递到`filterB`中

过滤器参数

   `{{ message | filterA('arg1', arg2) }}` 这个例子中，`filterA`被定义为接收三个参数的过滤器函数，其中`message`的值作为第一个参数，普通字符串`arg1` 作为第二个参数，表达式`arg2`的值作为第三个参数

## v-if，v-show, v-html原理

* `v-if`会调用`addIfCondition`,生成vnode的时候会忽略对应节点，render的时候就不会渲染
* `v-show`会生成vnode，render的时候也会渲染成真实的节点，只是在render过程中会在节点属性中修改show属性，也就是常说的display
* `v-html`会先移除节点下的所有节点，调用html方法，通过`addProp`添加`innerHTML`，归根到还是设置innerHTML为v-html的值

## v-show和v-if的区别

手段：

* `v-if`是动态的向dom树添加或删除domy元素
* `v-show`通过设置dom元素的display样式属性来控制显示隐

编译过程：

* `v-if`切换有一个局部编译/卸载的过程，切换的过程中合适的销毁和重构内部的事件监听和子组件
* `v-show`只是简单的基于css切换

编译条件：

* `v-if`是惰性的，如果初始条件为假，则什么都做；只有在条件第一次为真时才开始局部编译
* `v-show`是在任何条件下，无论首次条件是否为真，都被编译，然后被缓存，而且dom元素保留

性能消耗：

* `v-if`有更高的切换消耗
* `v-show`有更高初始渲染消耗

使用场景：

* `v-if`适合运营条件不太可能改变
* `v-show`适合平凡切换

## v-model是如何实现的，语法糖是什么

* 作用在表单元素上 动态绑定了input的value指向message变量，并且触发input事件的时候去动态把message设置为目标值

  ```html
    <input v-model="sth" />
    <!-- 等同于 -->
    <input 
      v-bind:value="message" 
      v-on:input="message=$event.target.value"
    >
    <!-- $event 指代当前触发的事件对象;
    $event.target 指代当前触发的事件对象的dom;
    $event.target.value 就是当前dom的value值;
    在@input方法中，value => sth;
    在:value中,sth => value; -->
  ```

* 作用在组件上 在自定义组件中，v-model默认会利用value的prop和名为input的事件

  本质上是一个父子组件通讯的语法糖， 通过prop和$emit实现。因此父组件v-model语法糖本质上可以修改为

  ```html

    <Child :value="message"  @input="function(e){message = e}"></Child>

  ```

  在组件的实现中，可以通过v-model属性来配置子组件接收prop名称，以及派发的事件名称

  ```html

    <!-- 父组件 -->
    <aa-input v-model="aa"></aa-input>
    <!-- 等价于 -->
    <aa-input v-bind:value="aa" v-on:input="aa=$event.target.value"></aa-input>

    <!-- 子组件： -->
    <input v-bind:value="aa" v-on:input="onmessage"></aa-input>
    <script>
      export default {
        props:{
          value: aa
        }
        methods:{
          onmessage(e){
            $emit('input', e.target.value)
          }
        }
      }
    </script>

  ```

默认情况下，一个组件的v-model会把value用作prop并且把input用作event。

但是一些输入类型（单选，复选）可能想使用value prop来达到不同目的。

使用v-model可以避免这些情况产生冲突。

js监听input输入框输入数据改变，用oninput， 数据改变后就会立即触发这个事件。

通过input事件把数据emit出去，在父组件接收。

父组件设置v-model的值为input emit过来的值
