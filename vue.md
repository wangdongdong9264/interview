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

js中对象是引用类型的数据，当多个实例引用一个对象时，只要一个实例对这个对象进行操作，其它实例中的数据也会发生变化

而vue中我们想要的是复用组件，那就需要每一个组件都有自己的数据，这样组件之间才不会相互干扰

所以组件的数据不能写成对象的形式，而是要写成函数的形式。数据以函数返回值的形式定义，
当这样我们每次复用组件的时候，就会返回一个新的data，也就是说每个组件都有自己的私有数据空间，它们各自维护自己的数据，不会干扰其它组件的正常运行

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

## 对keep-alive的理解，它是如何实现的，具体缓存的是什么

如果需要在组件切换的时候，保存一些组件的状态防止多次渲染，就可以使用keep-alive组件包裹需要保存的组件

`keep-alive`三个属性：

* `include` 字符串或正则表达式，只有名称匹配的组件会被缓存
* `exclude` 字符串或正则表达式，任何名称匹配的组件都不会被缓存
* `max` 数字，最多可以缓存多少组件实例

  keep-alive包裹动态组件时，会缓存不活动的组件实例

主要流程：

  1. 判断组件的name，不在include或exclude中，直接返回vnode，说明该组件不被缓存
  2. 获取组件实例的key
  3. key生成规则，cid+"::"+tag, 仅靠cid是不够的，因为相同的构造函数可以注册为不同的本地组件
  4. 如果缓存对象存在，则直接从缓存对象中获取组件实例给vnode，不存在则添加到缓存对象中
  5. 最大缓存数量，当缓存组件数量超过max值时，清除keys数组内第一个组件

步骤总结：

  1. 获取keep-alive下的第一个子组件的实例对象，通过它来获取组件的名称
  2. 通过当前的组件名去匹配原来include 和 enclude，判断当前组件是否需要缓存，不需要缓存则直接返回当前组件的vnode
  3. 需要缓存时，需要判断组件是否在缓存数组keys里面
     1. 存在，则将原来的key移除，同时将这个组件的key放到数组最后面（LRU）
     2. 不存在 将数组的key放入数组，然后判断当前keys数组是否超过mac所设置的范围，超过则删除未使用时间最长的一个组件key
  4. 最后将这个组件的keepAlive属性设置为ture

其它：

  问：keep-alive都知道，它不会生成真正的DOM节点，这是怎么做到的？
  答：vue在初始生命周期的时候，为组件实例建立父子关系会根据`abstract`属性决定是否忽略某个组件。在`keep-alive`中, 设置了`abstract: true`, 那vue就会跳过该组件实例

  问：keep-alive包裹的组件是如何使用缓存的？
  答：在patch阶段，会执行createComponent函数

  ```js
  // src/core/vdom/patch.js
  function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
    let i = vnode.data
    if (isDef(i)) {
      const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
      if (isDef(i = i.hook) && isDef(i = i.init)) {
        i(vnode, false)
      }
      if (isDef(vnode.componentInstance)) {
        initComponent(vnode, insertedVnodeQueue)
        insert(parentElem, vnode.elem, refElem) // 将缓存的DOM(vnode.elem) 插入父元素中
        if (isTrue(isReactivated)) {
            reactivateComponent(vnode, insertedVnodeQueue, parentEle, refElm)
        }
        return true
      }
    }
  }
  ```

  首次加载被包裹组件时，由keep-alive。js中的render函数可知，`vnode.componentInstance`的值是undefined，keepAlive的值是true，因为keep-alive组件为父组件，它的render函数会先于被包裹组件执行。那么只执行到`i(vnode, false)`, 后面的逻辑执行

  再次访问被包裹组件时，`vnode.componentInstance`的值就是已经缓存的组件实例，会执行`insert(parentElem, vnode.elem, refElem)`逻辑，这样就把上一次的dom插入到父元素中

  问： 一般的组件，每一次加载都会有完整的生命周期，为什么被keep-alive包裹的组件却不是呢
  答： 被缓存的组件实例会为其设置keepAlive=true，而在初始化组件钩子函数中

  ```js
    // src/core/vdom/create-component.js
    const componentVNodeHooks = {
      init (vnode: VNodeWithData, hydrating: boolean): ?boolean{
        if (
        vnode.componentInstance &&       
        !vnode.componentInstance._isDestroyed &&
        vnode.data.keepAlive
        ) {
          // keep-alive components, treat as a patch
          const mountedNode:any = vnode
          componentVNodeHooks.prepatch(mountedNode, mountedNode)
        } else {
          const child = vnode.componentInstance = createComponentInstanceForVnode (vnode, activeInstance)
        }
      }
    }
  ```

  可以看出，当`vnode.componentInstance`和`keepAlive`同时为true时，不再进入$mount过程，那mounted之前的所有钩子函数（beforeCreate、created、mounted）都不再执行

## $nextTick 原理及作用

vue的`nextTick`其本质是对javascript执行原理 evenloop的一种应用

nextTick的核心是利用如promise/MutationObserver/setImmediate/setTimeout的原生js方法来模拟宏/微任务的实现，本质是为了利用js的这些异步回调任务队列来实现vue框架中自己的异步回调队列

nextTick不仅是vue内部异步队列的调用方法，同时也允许开发者在实际项目中使用这个方法来满足对dom更新数据后的逻辑处理

nextTick是典型的将js执行原理应用到具体案例中的示例，引入异步更新队列机制的原因

  1. 如果是同步更新，则多次对一个或多个属性赋值，会频繁触发ui/dom渲染，可以减少一些无用的渲染
  2. 由于`VirtualDOM`的引用，每一次状态发生变化后，状态变化的信号会发送给组件，组件内部使用VirtualDOM进行计算得出具体需要更新的dom节点，然后对dom进行更新操作，每次更新状态后的渲染过程需要更多的计算，这种无用功也将浪费更多的性能，所以异步渲染变得更加重要

nextTick的使用场景

  1. 在数据变化后执行的某个操作，而这个操作需要数据变化而变化的dom结构的时候，这个操作就需要在nextTick的回调函数中
  2. 在vue生命周期中，如果created钩子进行dom操作（因为在created钩子函数中，页面的DOM还未渲染，这时候也没办法操作DOM），也一定要放在nextTick的回调函数中

## Vue 中给 data 中的对象属性添加一个新的属性时会发生什么？如何解决？

 ```html
  <template> 
    <div>
        <ul>
          <li v-for="value in obj" :key="value"> {{value}} </li> 
        </ul> 
        <button @click="addObjB">添加 obj.b</button> 
    </div>
  </template>
  <script>
    export default { 
      data () { 
        return { 
          obj: { 
            a: 'obj.a' 
          } 
        } 
      },
      methods: { 
        addObjB () { 
          this.obj.b = 'obj.b' 
          console.log(this.obj)
        } 
      }
    }
  </script>
 ```

点击 button 会发现，obj.b 已经成功添加，但是视图并未刷新。

这是因为在Vue实例创建时，obj.b并未声明，因此就没有被Vue转换为响应式的属性，自然就不会触发视图的更新，这时就需要使用Vue的全局api `$set()`

 ```js

  addObjB () {
    this.$set(this.obj, 'b', 'obj.b')
    console.log(this.obj)
  }

 ```

## vue data中某个属性的值发生改变后，视图会立即同步执行重新渲染吗？

不会立即执行同步重新渲染。

vue实现响应响应式并不是数据发生改变后dom立即变化，而是按照一定的策略进行dom更新。vue在更新dom是异步执行的。只要侦测到数据变化，vue将开启一个队列，并缓冲在同一个事件循环中发生的所有数据变更

如果同一个watcher被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和dom操作是非常重要的。然后在下一个事件循环`tick`中，vue刷新队列并执行（已经去重）数据

## vue单页应用和多页应用的区别

概念：

* spa单页面，指只有一个页面的主应用，一开始只需要加载一次js，css等相关资源，所有内容都包含在主页面，对每一个功能模块组件化。单叶应用跳转，就是切换相关组件，仅仅刷新局部资源。
* mpa多页面，指多个独立页面的应用，每个页面必须重复加载js，css等相关资源。多页面应用跳转，需要整页资源刷新

区别：

| 对比/模式      | spa | mpa  |
| :---        |    :----:   |    :----:   |
| 结构      | 一个主页面+许多模块组件       | 许多完整的页面   |
| 体验      | 页面切换快，体验佳；       | 页面切换慢   |
| 资源文件      | 组件公共资源只需要加载一次       | 每个页面都要有自己加载公用的资源   |
| 适用场景      | 对体验度和流畅度有较高要求的应用       | 适用于对seo要求较高的应用   |
| 内容更新      | 局部更新       | 整体html的切换   |
| 路由模式      | hash/history模式       | 普通链接跳转   |
| 数据传递      | 一般vuex       | cookie/url参数   |
| 开发成本      | 前期成本高，后期易于维护       | 前期开发成本低，后期维护难   |
