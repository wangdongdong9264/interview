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
