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
