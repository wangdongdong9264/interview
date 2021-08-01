# react

## react事件绑定原理

```js

<div onClick={this.handleClick.bind(this)}>点我</div>

```

react并不是将`click`事件绑定到了div的真实dom上，而是在`document`处监听了所有的事件，当事件发生并且冒泡到document处的时候，react将事件内容封装并交由真正的处理函数运行。这样的方式不仅仅减少了内存的消耗，还能在组件挂载/销毁时统一订阅/移除事件

除此之外，冒泡到document上的事件也不是原生的dom事件，而是由react自己实现的合成事件`SyntheticEvent`。因此如果不想要事件冒泡的话应该调用`event.preventDefault()` 而不是调用`event.stopProppagation()`

## react的事件和普通事件的html事件有什么不同

区别：

  1. 对于事件名称命名方式，原生事件为全小写，react采用小驼峰
  2. 对于事件函数处理语法，原生事件为字符串，react为事件函数
  3. react不能通过`return false`的方式来阻止浏览器默认行为，而必须要明确的调用`event.preventDefault()`来阻止默认行为

合成事件是react模拟原生dom事件所有能力的一个事件对象，有点如下：

  1. 兼容所有浏览器，更好的跨平台
  2. 将事件统一放在一个数组，避免频繁的新增和删除（垃圾回收）
  3. 方便react统一管理和事务机制

事件的执行顺序为原生事件先执行，合成事件后执行，合成事件会冒泡绑定到document上，所以尽量避免原生事件与合成事件混用，如果原生事件阻止冒泡，可能会导致合成事件不执行，因为需要冒泡到document上合成事件才会执行

## react组件中怎么做事件代理？他的原理是什么?

react基于`Virtual DOM`实现了一个`SyntheticEvent`层（合成事件层），定义的事件处理器会接收到一个合成事件对象的实例，它符合w3c标准，且与原生的浏览器事件拥有相同的接口，支持冒泡机制，所有的事件都自动绑定在最外层上。

在react底层，主要对合成事件做了： 事件委派和自动绑定。

* 事件委派：react会把所有的事件绑定到结构的最外层，使用统一的事件监听器，这个事件监听器上维持了一个映射来保存所有数组内部事件监听和处理函数
* 自动绑定：react组件中，每个方法的上下文都会指向该组件的实例，即自动绑定this为当前组件

## 对React-Fiber的理解，它解决了什么问题？

react 15.x版本在渲染时，会递归对比虚拟dom树，找到需要变动的节点，然后同步更它们。这个过程期间，react会占据浏览器资源，这会导致用户触发的事件得不到响应，并且会导致掉帧，用户感觉到卡顿

为了给用户制造一种应用很快的假象，不能让一个人长时间霸占资源。可以将浏览器的渲染，布局，绘制，资源加载（例如html解析），事件响应，脚本执行视作操作系统的“进程”，需要通过某些调度策略合理的分配cpu资源，从而提高浏览器的用户响应速率，同时兼顾任务执行效率

所以react通过`Fiber`架构，让这个执行过程变成可被中断。合理的让出cpu执行权，除了可以让浏览器及时的响应用户交互，还有如下优点：

1. 分批延时对dom进行操作，避免一次操作大量dom节点，可以得到更好的用户体验
2. 给浏览器一点喘息的机会，它会对代码进行编译优化（jit）及进行热代码优化，或者对reflow进行修正

核心思想：`fiber`也叫协程。它和线程并不一样，协程本身是没有并发或者并行能的（需要配合线程），它只是一种控制流程的让出机制。让出cpu的执行权，让cpu能在这段时间执行其它操作。渲染的过程可以被中断，可以将控制权交回给浏览器，让位给高优先级的任务，浏览器空闲后再恢复渲染

## `React.Component`和`React.PureComponent`的区别

PureComponent 表示一个纯组件，可以用来优化react程序，减少render函数执行的次数，从而提高组件性能

react中，当props或state发生变化时，可以通过 shouldComponentUpdate 生命周期函数中执行return false来阻止页面更新，从而减少不必要的render执行。PureComponent会自动执行 shouldComponentUpdate 

PureComponent 中的 `shouldComponentUpdate()`进行的是浅比较，也就是说如果是引用类型的数据，自会比较是不是同一地址，不会比较数据是否一致。浅比较会忽略属性和状态的突变情况，其实也就是指针没有变化，而数据发生改变的时候render是不会执行的。如果需要重新渲染那就需要重新开辟空间引用数据。PureComponent 一般会用在一些纯展示组件上

使用 PureComponent 的好处: 当组件更新时，如果组件的props或者state都没有改变，render函数就不会触发。省去了虚拟dom的生成和对比过程，达到提升性能的目的。这是因为react自动做了一层浅比较

## Component, Element, Instance之间有什么区别和联系

元素：一个元素 `element`是一个普通对象（plain object），描述了对于一个dom节点或者其他组件`component`, 你想让它在屏幕上呈现成什么样子。元素`element`可以在它的属性`props`中包含其它元素（用于形成元素树）。创建一个react元素elemen成本很低。元素element创建后是不可改变的

组件：一个组件`component`可以通过多种方式声明。可以是带有一个`render()`方法的类，简单点也可以定义为一个函数。这两种情况下，它都把属性`props`作为输入，把返回的一颗元素树作为输出

实例：一个实例`instance`是你在所写的组件类`component class`中使用关键字`this`所指向的东西（组件实例）。它用来存储本地状态和响应生命周期事件很有用

函数式组件（functional component）根本没有实例`instance`。
类组件（class component）有实例`instance`, 但是永远也不需要直接创建一个组件的实例，应为react帮我们做了这些

## React.createClass和extends Component的区别有哪些？

主要区别:

（1）语法区别

* createClass本质上是一个工厂函数，extends的方式更加接近新的es6规范class写法。两种方式在语法上的差别主要体现在方法的定义和静态属性的声明上。

* createClass方式的方法定义使用逗号，隔开，因为creatClasss本质上是一个函数，传递给它的是个object; 而class的方式定义方法时务必谨记不要使用都逗号隔开，这是es6 class的语法规范。

（2）propType和getDefaultProps

* React.createClass：通过propTypes对象和getDefaultProps()方法来设置和获取props

* React.Component: 通过设置两个属性propTypes和defaultProps

（3）状态的区别

* React.createClass: 通过getInitialState()方法返回一个包含初始值的对象
* React.Component： 通过constructor设置初始状态

（4）this区别

* React.createClass: 会正确绑定this
* React.Component: 由于使用了es6， 这里会有些不同，属性并不会自动绑定到React类的实例上

（5）Mixins

* React.createClass: 使用React.createClass的话，可以在创建组件时添加一个叫做mixins的属性，并将可混合的类的集合以数组的形式赋给mixins
* 如果使用es6的方式来创建组件，那么`react mixins`的特性将不能被使用

## react高阶组件是什么，和普通组件有什么区别，适用什么场景

官方解释:

  高阶组件（HOC）是react中用与复用组件逻辑的一种高级技巧。HOC自身不是react api的一部分，它是一种基于react的组合特性而形成的设计模式

高阶组件（HOC）就是一个函数，该函数接手一个组件作为参数，并返回一个新的组件，它只是一种组件的设计模式，这种设计模式是由react自身的组合性质必然产生的。我们将它们称为纯组件，因为它们可以接受任何动态提供的子组件，
但它们不会修改或复制其输入组件中的任何行为

```js

// hoc的定义
function withSubscription(WrappedComponent, selectData) {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        data: selectData(DataSource, props)
      };
    }
    // 一些通用的逻辑处理
    render() {
      // ... 并使用新数据渲染被包装的组件!
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };

// 使用
const BlogPostWithSubscription = withSubscription(BlogPost, (DataSource, props) => DataSource.getBlogPost(props.id));

```

HOC的优缺点

  优点：逻辑复用，不影响被包裹组件的内部逻辑
  缺点：HOC传递给包裹组件的props容易和被包裹后的组件重名，进而被覆盖

适用场景

  * 代码服复用，逻辑抽象
  * 渲染劫持
  * state 抽象和更改
  * Props 更改

具体应用
  
1. 权限控制：利用高阶组件的`条件渲染`特性可以对页面进行权限控制，权限控制一般分为两个维度：页面级别和页面元素级别

2. 组件渲染性能追踪：借助父子组件
生命周期规则捕获子组件的生命周期，可以方便的对某个组件的渲染时间进行记录

3. 页面复用

## 对componentWillReceiveProps的理解

该方法当`props`发生变变化时执行，初始化`render`时不执行，在这个回调函数里面，你可以根据属性的变化，通过调用`this.setState()`来更新你的组件状态，旧的属性还是可以通过`this.props`来获取，这里调用更新状态是安全的，并不会触发额外的`render`调用

使用好处：在这个生命周期中，可以在子组件的render函数执行前获获取新的props，从而更新子组件自己的state。可以将数据请求放在这里进行执行，需要传的参数则从`componentWillReceiveProps(nextProps)`中获取。而不必将所有的请求都放在父组件中。于是该请求只会在该组件渲染时才会发出，从而减轻请求负担。

`componentWillReceiveProps`在初始化render的时候不会执行，它回在Component接受到新的状态（Props）时被触发，一般用于父组件状态更新时子组件的重新渲染

## 哪些方法会触发React重新渲染？重新渲染render会做些什么

（1）哪些方法会触发react重新渲染

`setState()`方法被调用

  setState是react中最常用的命令，通常情况下，执行setState会触发render。但是这里有个点值得关注，执行setState的时候不一定会重新渲染。当setState传入null时，并不会触发render

父组件重新渲染

只要父组件重新渲染了，即使转入子组件的props未发生变化，那么子组件也会重新渲染，进而触发render


（2）重新渲染render会做些什么？

  * 会对新旧VNode进行对比，也就是我们所说的DiffDiff算法
  * 对新旧两棵树进行一个深度优先遍历，这样每个节点都会有一个标记，在到深度遍历的时候，每遍历到一个节点，就把该节点和新节点树进行对比，如果有差异就放到一个对象里面
  * 遍历差异对象，根据差异的类型，对应规则更新VNode

react 的处理render的基本思维模式是每一次有变动就会去重新渲染整个应用。在`Virtual DOM`没有出现之前，最简单的方式就是直接调用`innerHTML`。Virtual Dom厉害的地方并不是说它不直接操作dom快，而是说不管数据怎么变，都会尽量以最小的代价去更新DOM。React将render函数返回的虚拟dom树与老的向比较，从而确定dom要不要更新，怎么更新。当dom树很大时，遍历两颗树进行各种比对还是相当消耗性能的，特别时在顶层setState一个微小的修改，默认会去遍历整个树。尽管React使用高度优化的`Diff`，但是这个过程仍然损耗性能

## react如何判断什么时候重新渲染组件？

组件状态的改变可以因为`props`的改变, 或者直接通过`setState`方法改变。组件获得新的状态，然后react决定是否应该重新渲染组件。只要组件的state发生变化，React就会对组件进行重新渲染。这是因为React中的`shouldComponentUpdate`方法默认返回`true`, 这就是导致每次更新都重新渲染的原因。

当React将要渲染组件实回执行`shouldComponentUpdate`方法来看它是否返回true（组件应该更新，也就是重新渲染）。所以需要重写`shouldComponentUpdate`方法让它根据情况返回`true`或者`false`来告诉react什么时候重新渲染什么时候跳过重新渲染

## react生声明组件有哪几种方法，有什么不同？

React 声明组件的三种方式：

1. 函数式定义的`无状态组件`
2. es5原生方式`React.createClass`定义的组件
3. es6形式的 `extends React.Component`

`无状态函数式组件`它是为了创建存展示组件，这种组件只负责根据传入的Props来展示，不涉及到state状态的操作 组件不会被实例化，整体渲染性能的到提升，不能访问this对象, 不能访问声明周期的方法

`es5原生方式 React.createClass` （RFC） 会自绑定函数方法，导致不必要的性能开销，增加代码过时的可能性。

`es6形式的 React.Component` （RCC）目前极为推荐的创建有状态组件的方式，最终取代React.createClass形式；相对于 React.createClass 可以更好实现代码复用

无状态组件相对于后者的区别：与无状态组件相比，`React.createClass`和`React.Component`都是创建有状态的组件，这些组件是要被实例化的，并且可以访问组件的生命周期方法

React.createClass与React.Component区别:
  函数this自绑定
    
    React.createClass创建的组件,其每一个成员函数的this都有react自动绑定，函数中的this会被正确设置。
    
    React.Component创建的组件,其成员函数不会自动绑定this, 需要开发者手动绑定，否则this不能获取当前组件实例对象

  组件属性类型propTypes及其默认porps属性defaultProps配置不同

    React.createClass在创建组件时，有关组件props的属性类型及组件默认的属性会作为组件实例的属性来配置，其中defaultProps的方法来获取默认组件属性的

    React.Component在创建组件时，配置这两个对信息时，它们时是作为组件类的属性，不是组件实例的属性，也就是所谓的类的静态属性来配置的

  组件初始状态state的配置不同

    React.createClass在创建组件时, 其状态state是通过getInitialState方法来配置组件相关的状态;

    React.Component在创建组件时，其状态state是在constructor中像初始化组件属性一样声明的

## 对有状态组件和无状态组件的理解及使用场景

有状态组件

特点：

  1. 是类组件
  2. 有继承
  3. 可以使用this
  4. 可以使用react的生命周期
  5. 使用较多，容易频繁触发生命周期钩子函数，影响性能
  6. 内容使用state，维护自身状态的变化，有状态组件根据外部组件传入的props和自身的state进行渲染

使用场景：

  1. 需要使用到状态
  2. 需要使用状态操作的（无状态组件的也可以实现新版本react hooks也可以实现）

总结：类组件可以维护自身的状态变量，即组件的state，类组件还有不同的生命周期方法，可以让开发者能够在组件的不同阶段（挂载，更新，卸载），对组件做更多的控制。类组件则即可以充当无状态组件，也可以充当有状态组件。当一个类组件不需要管理自身状态时，也可称未无状态组件

---

无状态组件

特点：

  1. 不依赖自身的状态state
  2. 可以时类组件或者函数组件
  3. 可以完全避免使用this关键字。（由于使用的是箭头函数事件无需绑定）
  4. 有更高的性能。当不需要使用生命周期钩子时，应该首先使用无状态函数组件
  5. 组件内部不维护state，根据外部组件传入的props进行渲染的组件，当props改变时，组件重新渲染

使用场景：

  1. 组件不需要管理state，纯展示

优点：

  1. 简化代码，专注render
  2. 组件不需要被实例化，无生命周期，提升性能。输出（渲染）只取决于输入（属性），无副作用
  3. 视图和数据的解耦分离

缺点： 

  1. 无法使用ref
  2. 无生命周期方法
  3. 无法控制组件的重渲染，因为无法使用`shouldComponentUpdate`方法，当组件接受到新的属性时则会重渲染

总结： 组件内部状态与外部无关的组件，可以考虑用状态组件，这样状态就不会过于复杂，易于理解和管理。当一个组件不需要管理自身状态时，也就是无状态组件, 应该优先设计为函数组件。比如自定义的`<Button/>`, `<Input/>`等组件。

## 对React中Fragment的理解，它的使用场景是什么

在React中，组件返回的元素只能有一个跟元素。为了不添加多余的DOM节点，我们可以使用`Fragment`标签来包裹所有的元素，Fragment标签不会渲染出任何元素。react官方对Fragment的解释

  react中的一个常见模式是一个组件返回多个元素。`Fragment`允许你将子列表分租，而无需向DOM添加额外的节点

```js

import React, { Component, Fragment } from 'react'

// 一般形式
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
// 也可以写成以下形式
render() {
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildC />
    </>
  );
}


```

## 如何获取组件对应的dom元素？

可以用ref来获取某个子节点的实例，然后通过当前class组件实例的一些特定属性来直接获取子节点实例。ref有三种实现方法：

  1. 字符串格式：字符串格式，这是React16版本之前用的最多， 例如`<p ref="info">span</p>`
  2. 函数格式： ref对应一个方法，该方法有一个参数，也就是对应的节点实例`<p ref={ele => this.info = ele}></p>`
  3. createRef方法：React 16提供的一个api，使用`React.createRef()`来实现

## React中可以在render访问refs吗？ 为什么？

```html

<>
  <span id="name" ref={this.spanRef}>{this.state.title}</span>
  <span>{
     this.spanRef.current ? '有值' : '无值'
  }</span>
</>

```

不可以,render阶段DOM还没有生成，无法获取DOM。DOM的获取需要在`pre-commit`阶段和`commit`阶段`

`render`阶段：纯净且不包含副作用。可能会被react暂停，中止或重新启动

`pre-commit`阶段：可以读取dom

`commit`阶段：可以使用dom，运行副作用，安排更新

## 对react的插槽（Portals）的理解，如何使用，有哪些使用场景

React官方对Portals的定义：

  Portal提供了一种子节点渲染到存在于父组件以外的DOM节点的优秀的方案

`Portals`是React 16提供的官方解决方案, 使得组件可以脱离父组件层级挂载在dom树的任何位置。通俗来讲，就是我们render一个组件，但这个组件的DOM结构并不在本组件内

`Portals`语法如下：

```js

ReactDOM.createPortal(child, container)

// 第一个参数child是可渲染的React子项，比如元素，字符串或者片段等
// 第二个参数container是一个DOM元素

```

一般情况下，组件的render函数返回的元素会被挂载在它的父级组件上：

```js
import DemoComponent from './DemoComponent';
render() {
  // DemoComponent元素会被挂载在id为parent的div的元素上
  return (
    <div id="parent">
        <DemoComponent />
    </div>
  );
}

```

然而有些元素需要挂载再更高层级的位置。最经典的应用场景: 当父级组件具有`overflow：hidden`或者`z-index`的样式设置时，组件有可能被其他元素遮挡，这时就可以考虑要不要使用`Portals`使组件的挂载脱离父组件。例如：对话框，模态窗

```js

import DemoComponent from './DemoComponent';
render() {
  // DemoComponent元素会被挂载在id为parent的div的元素上
  return (
    <div id="parent">
        <DemoComponent />
    </div>
  );
}

```

## 再React中如何避免不必要的render?

React 基于虚拟dom和高效Diff算法的完美配合, 实现了对dom最小颗粒度的更新。大多数情况下，React对DOM的渲染效率足以业务日常。但在个别复杂业务场景下，性能问题依然会困扰我们。此时就需要采取一些措施来提升运行性能，其很重要的一个方向，就是避免不必要的渲染（Render）。这是提下优化的点

`shouldComponentUpdate`和`PureComponent`

  在react类组件中，可以利用`shouldComponentUpdate`或者`PureComponent`来减少因父组件更新而触发子组件的render，从而达到目的。`shouldComponentUpdate`来决定是否重新渲染，如果不希望组件重新渲染，返回false即可

利用高阶组件

  在函数组件库中，并没有`shouldComponentUpdate`这个生命周期，可以利用高阶组件，封装一个类似`PureComponent`的功能

使用`React.memo`

React.memo是React 16.6新增的一个api，用来缓存组件的渲染，避免不必要的更新，其实也是一个高阶组件，与`PureComponent`十分类似，但是不同的是，React.memo只能用于函数组件

## 对React-Intl的理解，它的工作原理？

`React-Intl`是雅虎的语言国际化开源项目FormatJS的一部分，通过其提供的组件和api可以与ReactJS绑定

`React-Intl`提供了两种使用方法，一种是引入React组件，另一种是直接调取api，官方更加推荐在react项目中使用前者，只有无法使用React组件的地方，才应该调用框架提供api。它提供了一系列的React组件，包括数字格式化，字符串格式化，日期格式化等。

在`React-Intl`中，可以配置不同的语言包，他的工作原理就是根据需要，在语言包之间进行切换。

## 对React context的理解

在React中，数据传递一般使用`props`传递数据，维持单项数据流，这样可以让组件之间的关系变得简单且可预测，但是单向数据流在某些场景中并不适用。单纯一对的父子传递并无问题，但要是组件之间层层依赖深入，props就需要层层传递，显然这样太繁琐

`Context`提供了一种在组件之间共享此类值的方式，而不必显式的通过组件树的逐层传递给props

可以把`context`当作是一个特定的组件树内共享的store，用来做数据传递。简单来说就是，当你不想在组件树中通过逐层传递props或者state的方式来传递数据时，可以使用Context来实现跨层级的组件数据传递。

js的代码块在执行期间，会创建一个相应的作用域链，这个作用域链纪录着代码块执行期间所能访问的活动对象，包括变量和函数，js程序通过作用域访问到代码块内部或者外部的变量和函数。

假设以js的作用域作为类比，React组件提供的Context对象其实就好比一个提供给子组件访问的作用域，而Context对象属性可以看成作用域上的活动对象。由于组件的Context由其父节点链上所有组件通过`getChildContext()` 返回Context对象组合而成，所以，组件通过Context是可以访问到其父组件链上所有节点组件提供的Context的属性。

## 为什么React并不推荐优先考虑使用Context?

1. Context目前还处于实验阶段，可能会在后面的发行版本中有很大变化，事实上这种情况已经发生了，所以为了避免给今后升级带来大的影响和麻烦，不建议在app中使用Context。

2. 尽管不建议在app中使用Context, 但是独有组件而言，由于影响范围小于app，如果可以做到高内聚，不破坏组件树之间的以来关系，可以考虑使用Context

3. 对于组之间的数据通讯或者状态管理，有效使用props或者state解决，然后再考虑使用第三方成熟库进行解决，以上的方法都不是最佳的方案的时候，在考虑Context

4. Context的更新需要通过`setState()`触发，但是这并不是很可靠的，Context支持跨组件的访问，但是如果中间的子组件通过一些方法不影响更新，比如`shouldComponentUpdate()`返回`false`那么不能保证Context的更新一定可以使用Context的子组件，因此，Context的可靠性需要关注

## React中什么是受控组件和非控组件

`受控组件`在使用表单来收集用户输入时，例如`<input>`,`<select>`,`<textearea>`等元素都要绑定一个change事件，当表单的状态发生变化，就会触发`onChange`事件，更新组件的state。这种组件在React中被称为`受控组件`，在受控组件中，渲染出来的状态与它的value或checked属性想对应，react通过这种方式消除了组件的局部状态，使整个状态可控。react官方推荐使用受控表单组件。

受控组件更新state的流程：

  1. 可以通过初始state中设置表单的默认值
  2. 每当表单的值发生变化时，调用onChange事件处理器
  3. 事件处理器通过事件对象e拿到改变后的状态，并更新组件的state
  4. 一旦通过setState方法更新state，就会触发视图的重新渲染，完成表单组件的更新

受控组件的缺点：

  表单元素的值都是由react组件进行管理，当有多个输入框，或则多个这种组件时，如果想同时获取到全部的值就必须每个都要编写事件处理函数，这会让代码看着臃肿，所以为了解决这种情况，出现了`非受控组件`

`非受控组件`如果一个表单组件没有value props（单/多选按钮对应的时 checked props）时，就可以称为非受控组件。在非受控组件中，可以使用一个`ref`来从DOM获得表单值。而不是为了每个状态更新编写一个事件处理程序。

React官方的解释：

  要编写一个非受控组件，而不是为了每个状态更新都编写数据处理函数，你可以使用ref来DOM节点中获取表单数据。因为非受控组件将真实数据存储在DO节点中，所以在使用非受控组件时，有时候反而更容易集成React和非React代码。如果你不建议代码的美观性，并且希望快速编写代码，使用非受控代码往往可以减少你的代码量。否则，你应该使用受控组件

```js

// 例如 代码在非受控组件中接收单个属性

class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }
  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }
  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}

```

总结: 页面中所有输入类的DOM如果是现用现取的称为非受控组件，而通过setState将输入的值维护到了state中，需要时再从state中取出，这里的数据就受到了state的控制，称为受控组件。

## react中refs的作用是什么？有哪些应用场景？

Refs提供了一种方式，用于访问在`render`方法中创建的react元素或DOM节点。refs应该谨慎使用，如下场景使用Refs比较合适:

  1. 处理焦点，文本选择或则媒体的控制
  2. 触发必要的动画
  3. 集成第三方DOM库

`Refs`是使用`React.createRef()`方法创建的， 它通过`ref`属性附加到react元素上。要在整个组件中使用refs，需要将ref在构造函数中分配给实例属性

```js

class MyComponent extends React.Component {
  constructor(props) {
    super(props)
    this.myRef = React.createRef()
  }
  render() {
    return <div ref={this.myRef} />
  }
}

```

由于函数组件没有实例，因此不能再函数组件上直接使用`ref`：

```js

function MyFunctionalComponent() {
  return <input />;
}
class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }
  render() {
    // 这将不会工作！
    return (
      <MyFunctionalComponent ref={this.textInput} />
    );
  }
}

```

但可以通过闭合的帮助再函数租几间内部进行使用`refs`:

```js

function CustomTextInput(props) {
  // 这里必须声明 textInput，这样 ref 回调才可以引用它
  let textInput = null;
  function handleClick() {
    textInput.focus();
  }
  return (
    <div>
      <input
        type="text"
        ref={(input) => { textInput = input; }} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );  
}

```

注意：

  1. 不应该过度使用refs
  2. `ref`的返回值取决于节点的类型：
    * 当`ref`属性被用于一个普通html元素时，`React.createRef()`将接收底层DOM元素为它的`current`属性以创建`ref`
    * 当`ref`属性被用于一个自定义的类组件时，`ref`对象将接受该组件已挂载的实例作为它的`current`。
  3. 当再父组件中需要访问子组件中的ref时可以使用传递Refs或则回调Refs。

## React组件的构造函数有什么作用？它是必须的吗？

构造函数主用于两个目的：

1. 通过将对象分配给this.state来初始化本地状态
2. 将事件处理程序方法绑定到实例上

所以，当在react class中需要设置state的初始值或者绑定事件时，需要加上构造函数，官方demo:

```js

class LikeButton extends React.Component {
  constructor() {
    super();
    this.state = {
      liked: false
    };
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    this.setState({liked: !this.state.liked});
  }
  render() {
    const text = this.state.liked ? 'liked' : 'haven\'t liked';
    return (
      <div onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </div>
    );
  }
}
ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);

``` 

构造函数用来新建父类的this对象；子类必须在constructor方法中调用super方法；否者新建实例时会报错；因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法；子类就得不到this对象。

注意：

1. `constructor()`必须配上`super()`，如果要在`constructor`内部使用`this.props`就要传入`props`，否则不用
2. javaScript中的bind每次都会返回一个新的函数，为了性能的考虑，尽量在`constructor`中绑定事件

## React.forwardRef是什么？它有什么作用？

`React.forwardRef`会创建一个React组件，这个组件能够将其接受的ref属性转发到其组件树下的一个组件中。这种技术并不常见，但在一下两种场景中特别有用：

1. 转发refs到DOM组件
2. 在高阶组件中转发refs

## 类组件于函数组件有什么异同？

相同点：

* 组件时react可复用的最小代码片段，它们会返回要在页面中渲染React元素。也正因为组件是react的最小编码单位，所以无论是函数组件还是组件，在使用方式和最终效果上都是1完全一致的。

* 我们甚至可以将一个类组件改写成函数组件，或者把函数组件改写成一个类组件（虽然不推荐）。从使用者的角度而言，很难从使用体验上区两者，而且现代浏览器中，闭包和类的性能只在极端场景下才会有明显的差别。所以，基本可以认为两者作为组件是完全一致的。

不同点：

* 它们在开发时的心智模型上却存在巨大差异。类组件是基于面向对象编程的，它主打的是继承，生命周期等核心概念；而函数组件内核是函数式编程，主打的是`immutable`，没有副作用，应用透明等特点。
* 之前，在使用场景上，如果存在需要使用生命周期的组件，那么主推类组件；设计模式上，如果需要使用继承，那么主推类组件。但是现在由于React Hooks的推出，生命周期概念的淡出，函数组件可以完全取代类组件。其次继承并不是组件最佳设计模式，官方更推崇"组合优于继承"的设计概念，所以类组件在这些方面的优势也在淡出。
* 性能优化上，类组件主要依靠`shouldComponentUpdate`阻断渲染来提升性能，而函数组件依靠`React.memo`缓存渲染结果来提升性能。
* 从上手程度而言，类组件更容易上手，从未来趋势上看，由于`React Hooks`的推出，函数组件成了社区未来主推的方案。
* 类组件在未来时间切片与并发模式中，由于生命周期带来的复杂度，并不易于优化。而函数组件本身轻量简单，且在`Hooks`的基础上提供了比原先更细颗粒度的逻辑组织与复用，更能适应React未来的发展

## React setState调用的原理

`setState` => `enqueueSetState` => `enqueueUpdate` => `isBatchingUpdates`

(false) => 循环更新`dirtyComponents`中的所有组件

（true）=> 组件入队`dirtyComponents`

具体执行过程如下：

* 首先调用`setState`入口函数，入口函数在这里就是充当一个分发器的角色，根据入参的不同，将其分发到不同的功能函数中去；

```js

ReactComponent.prototype.setState = function (partialState, callback) {
  this.updater.enqueueSetState(this, partialState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
};

```

* `enqueueSetState`方法将新的`state`放进组件的状态队列里，并调用`enqueueUpdate`来处理将要更新的实例对象

```js

enqueueSetState: function (publicInstance, partialState) {
  // 根据 this 拿到对应的组件实例
  var internalInstance = getInternalInstanceReadyForUpdate(publicInstance, 'setState');
  // 这个 queue 对应的就是一个组件实例的 state 数组
  var queue = internalInstance._pendingStateQueue || (internalInstance._pendingStateQueue = []);
  queue.push(partialState);
  //  enqueueUpdate 用来处理当前的组件实例
  enqueueUpdate(internalInstance);
}

```

* 在`enqueueUpdate`方法中引出了一个关键的对象`batchingStrategy`，该对象所具备的`isBatchingUpdates`属性直接决定了当下是要走更新流程，还是应该排队等待；如果轮到执行，就调用`batchedUpdates`方法来直接更新流程。由此可以推测，`batchingStrategy`正是react内部专门用于管控批量更新的对象

```js

function enqueueUpdate(component) {
  ensureInjected();
  // 注意这一句是问题的关键，isBatchingUpdates标识着当前是否处于批量创建/更新组件的阶段
  if (!batchingStrategy.isBatchingUpdates) {
    // 若当前没有处于批量创建/更新组件的阶段，则立即更新组件
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }
  // 否则，先把组件塞入 dirtyComponents 队列里，让它“再等等”
  dirtyComponents.push(component);
  if (component._updateBatchNumber == null) {
    component._updateBatchNumber = updateBatchNumber + 1;
  }
}

```

注意：`batchingStrategy`对象可以理解为`锁管理器`（这里的锁指的是react全局唯一的`isBatchingUpdates`变量）。`isBatchingUpdates`的初始值是`false`，意味着`当前并未进行任何批量更新操作`。每当react调用`batchedUpdate`去执行更新动作时，会先把`isBatchingUpdates`置为`true`，表明`现在正处于批量更新的过程中`。当锁被锁上时，任何需要更新的组件都只能暂时进入`dirtyComponents`里排队等候下一次的批量更新，而不能随意插队。此处体现的`任务锁`的思想，是react面对大量状态仍然能够实现有序分批处理的基石

## React setState 调用之后发生了什么？是同步还是异步？

React中setState后发生了什么

  在代码中调用`setState`函数之后，react会将传入的参数对象与组件当前的状态合并，然后触发调和过程（Reconciliation）。经过调和过程，React会以相对高效的方式根据新的状态构建React元素树并且着手重新渲染整个UI界面

  在react得到元素树之后，react会自动计算出新的树与老树的节点差异，然后根据差异对界面进行最小化重渲染。在差异计算算法中，react能够相对精准的知道那些位置发生了改变以及应该如何改变，这就保证了按需更新，而不是全部重新渲染。

  如果在短时间内频繁`setState`。react会将state的改变压入栈中，在合适的时机，批量更新state和视图，达到提高性能的效果。

setState 是同步还是异步的

  假如说有`setState`是同步的，意味者每执行一次`setState`时，都需要重新vnode diff + dom 修改，这对性能来说是极为不好的。如果是异步，则可以把一个同步代码中多个`setState`合并成一次组件更新。所以默认是异步的，但是在一些情况下是同步的。

  `setState`并不是单纯同步/异步，它的表现会因调用场景不同而不同。在源码中，通过`isBatchingUpdates`来判断`setState`是先存进state队列还是直接更新，如果为true则执行异步操作，为false则直接更新。

  * 异步：在 React 可以控制的地方，就为 true，比如在 React 生命周期事件和合成事件中，都会走合并操作，延迟更新的策略。
  * 同步：在 React 无法控制的地方，比如原生事件，具体就是在 addEventListener 、setTimeout、setInterval 等事件中，就只能同步更新。

一般认为做异步设计是为了性能优化，减少渲染次数：

  `setState`设计为异步，可以显著的提升性能。如果每次调用 setState都进行一次更新，那么意味着render函数会被频繁调用，界面重新渲染，这样效率是很低的；最好的办法应该是获取到多个更新，之后进行批量更新

  如果同步更新了state，但是还没有执行render函数，那么state和props不能保持同步。state和props不能保持一致性，会在开发中产生很多的问题；

## React中的setState批量更新的过程是什么？

调用 setState 时，组件的 state 并不会立即改变， setState 只是把要修改的 state 放入一个队列， React 会优化真正的执行时机，并出于性能原因，会将 React 事件处理程序中的多次React 事件处理程序中的多次 setState 的状态修改合并成一次状态修改。 最终更新只产生一次组件及其子组件的重新渲染，这对于大型应用程序中的性能提升至关重要。

```js

this.setState({
  count: this.state.count + 1    ===>    入队，[count+1的任务]
});
this.setState({
  count: this.state.count + 1    ===>    入队，[count+1的任务，count+1的任务]
});
                                          ↓
                                         合并 state，[count+1的任务]
                                          ↓
                                         执行 count+1的任务


```

需要注意的是，只要同步代码还在执行，“攒起来”这个动作就不会停止。（注：这里之所以多次 +1 最终只有一次生效，是因为在同一个方法中多次 setState 的合并动作不是单纯地将更新累加。比如这里对于相同属性的设置，React 只会为其保留最后一次的更新）。

## React中有使用过getDefaultProps吗？它有什么作用？

通过实现组件的`getDefaultProps`，对属性设置默认值（ES5的写法）：

```js

var ShowTitle = React.createClass({
  getDefaultProps:function(){
    return{
      title : "React"
    }
  },
  render : function(){
    return <h1>{this.props.title}</h1>
  }
});

```

## React中setState的第二个参数作用是什么？

`setState` 的第二个参数是一个可选的回调函数。这个回调函数将在组件重新渲染后执行。等价于在`componentDidUpdate` 生命周期内执行。通常建议使用 componentDidUpdate 来代替此方式。在这个回调函数中你可以拿到更新后 state 的值:

```js

this.setState({
    key1: newState1,
    key2: newState2,
    ...
}, callback) // 第二个参数是 state 更新完成后的回调函数

```

## React中的setState和replaceState的区别是什么？

`setState()`用于设置状态对象，其语法如下:

```js

setState(object nextState[, function callback])

// nextState，将要设置的新状态，该状态会和当前的state合并
// callback，可选参数，回调函数。该函数会在setState设置成功，且组件重新渲染后调用

// 合并nextState和当前state，并重新渲染组件。setState是React事件处理函数中和请求回调函数中触发UI更新的主要方法。
```

`replaceState()`方法与setState()类似，但是方法只会保留nextState中状态，原state不在nextState中的状态都会被删除。其语法如下：

```js

replaceState(object nextState[, function callback])

// nextState，将要设置的新状态，该状态会替换当前的state。
// callback，可选参数，回调函数。该函数会在replaceState设置成功，且组件重新渲染后调用。

```

总结： `setState`是修改其中的部分状态了，相当于`Object.assign`只是覆盖，不会减少原来的状态，而`replaceState`是完全替换原来的状态，相当于赋值，将原来的state对象替换为另一个对象，如果新状态属性减少，那么state中就没有这个状态了

## 在React中组件的this.state和setState有什么区别？

this.state通常是用来初始化state的，this.setState是用来修改state值的。如果初始化了state之后再使用this.state，之前的state会被覆盖掉，如果使用this.setState，只会替换掉相应的state值。所以，如果想要修改state的值，就需要使用setState，而不能直接修改state，直接修改state之后页面是不会更新的。

## state 是怎么注入到组件的，从 reducer 到组件经历了什么样的过程

通过`connect`和`mapStateToProps`将state注入到组件中：

```js

import { connect } from 'react-redux'
import { setVisibilityFilter } from '@/reducers/Todo/actions'
import Link from '@/containers/Todo/components/Link'

const mapStateToProps = (state, ownProps) => ({
    active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
    setFilter: () => {
        dispatch(setVisibilityFilter(ownProps.filter))
    }
})

export default connect(
    mapStateToProps,
    mapDispatchToProps
)(Link)

```

上代码中，active就是注入到Link组件中的状态。`mapStateToProps = (state, ownProps)`两个参数的含义：

* `state`管理的全局状态对象，所有的组件状态数据都存储在该对象中。
* `ownProps`组件通过props传入的参数。

reducer 到组件经历的过程：

* `reducer`对`action`对象处理，更新组件状态，并将新的状态值返回store。
* 通过`connect（mapStateToProps，mapDispatchToProps）（Component）`对组件Component进行升级，此时状态值从store取出并作为props参数传递到组件。

高阶组件实现源码∶

```js

import React from 'react'
import PropTypes from 'prop-types'

// 高阶组件 contect 
export const connect = (mapStateToProps, mapDispatchToProps) => (WrappedComponent) => {
    class Connect extends React.Component {
        // 通过对context调用获取store
        static contextTypes = {
            store: PropTypes.object
        }

        constructor() {
            super()
            this.state = {
                allProps: {}
            }
        }

        // 第一遍需初始化所有组件初始状态
        componentWillMount() {
            const store = this.context.store
            this._updateProps()
            store.subscribe(() => this._updateProps()); // 加入_updateProps()至store里的监听事件列表
        }

        // 执行action后更新props，使组件可以更新至最新状态（类似于setState）
        _updateProps() {
            const store = this.context.store;
            let stateProps = mapStateToProps ?
                mapStateToProps(store.getState(), this.props) : {} // 防止 mapStateToProps 没有传入
            let dispatchProps = mapDispatchToProps ?
                mapDispatchToProps(store.dispatch, this.props) : {
                                    dispatch: store.dispatch
                                } // 防止 mapDispatchToProps 没有传入
            this.setState({
                allProps: {
                    ...stateProps,
                    ...dispatchProps,
                    ...this.props
                }
            })
        }

        render() {
            return <WrappedComponent {...this.state.allProps} />
        }
    }
    return Connect
}

```

## React组件的state和props有什么区别？

props:

  `props`是一个从外部传进组件的参数，主要作为就是从父组件向子组件传递数据，它具有可读性和不变性，只能通过外部组件主动传入新的props来重新渲染子组件，否则子组件的props以及展现形式不会改变。

state：

  `state`的主要作用是用于组件保存、控制以及修改自己的状态，它只能在`constructor`中初始化，它算是组件的私有属性，不可通过外部访问和修改，只能通过组件内部的`this.setState`来修改，修改state属性会导致组件的重新渲染。

区别：

1. `props`是传递给组件的（类似函数的参数），而state 是在组件内被组件自己管理的（类似于在一个函数内声明的变量）。
2. `props`是不可修改的，所有 React 组件都必须像纯函数一样保护它们的 props 不被更改
3. `state`是在组件中创建的，一般在`constructor`中初始化state。`state`是多变的、可以修改，每次setState都异步更新的。

## React中的props为什么是只读的？

`this.props`是组件之间沟通的一个接口，原则上来讲，它只能从父组件流向子组件。react具有浓重的函数式编程思想

提到函数式编程就要提一个概念`纯函数`特点：

1. 给定相同的输入，总是返回相同的输出。
2. 过程没有副作用
3. 不依赖外部状态

`this.props`就是吸取了纯函数的思想。props的不可变性就保证了相同的输入，页面显示的内容是一样的，并且不会产生副作用

## 在React中组件的props改变时更新组件的有哪些方法？

在一个组件传入的props更新时重新渲染该组件常用的方法是在`componentWillReceiveProps`中将新的props更新到组件的state中（这种state被成为派生状态（Derived State）），从而实现重新渲染。React 16.3中还引入了一个新的钩子函数`getDerivedStateFromProps`来专门实现这一需求。

`componentWillReceiveProps`（已废弃）:

  在react的`componentWillReceiveProps(nextProps)`生命周期中，可以在子组件的render函数执行前，通过this.props获取旧的属性，通过nextProps获取新的props，对比两次props是否相同，从而更新子组件自己的state。
  
  这样的好处是，可以将数据请求放在这里进行执行，需要传的参数则从`componentWillReceiveProps(nextProps)`中获取。而不必将所有的请求都放在父组件中。于是该请求只会在该组件渲染时才会发出，从而减轻请求负担。

`getDerivedStateFromProps`（16.3引入）:

  这个生命周期函数是为了替代`componentWillReceiveProps`存在的，所以在需要使用`componentWillReceiveProps`时，就可以考虑使用  `getDerivedStateFromProps`来进行替代。
  两者的参数是不相同的，而`getDerivedStateFromProps`是一个静态函数，也就是这个函数不能通过this访问到class的属性，也并不推荐直接访问属性。而是应该通过参数提供的`nextProps`以及`prevState`来进行判断，根据新传入的props来映射到state。

  需要注意的是，如果props传入的内容不需要影响到你的state，那么就需要返回一个null，这个返回值是必须的，所以尽量将其写到函数的末尾：

```js

static getDerivedStateFromProps(nextProps, prevState) {
    const {type} = nextProps;
    // 当传入的type发生变化的时候，更新state
    if (type !== prevState.type) {
        return {
            type,
        };
    }
    // 否则，对于state不进行任何操作
    return null;
}

```

## React中怎么检验props？验证props的目的是什么？

React为我们提供了`PropTypes`以供验证使用。当我们向Props传入的数据无效（向Props传入的数据类型和验证的数据类型不符）就会在控制台发出警告信息。它可以避免随着应用越来越复杂从而出现的问题。并且，它还可以让程序变得更易读。

```js

import PropTypes from 'prop-types';

class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};

```

当然，如果项目汇中使用了`TypeScript`，那么就可以不用PropTypes来校验，而使用TypeScript定义接口来校验props。

## React的生命周期有哪些？
