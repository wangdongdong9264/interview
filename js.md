# js

## promise

  `Promise.all()`
    只要任何一个输入的promise的reject回调执行或者输入不合法的promise就会立即抛出错误，并且reject的是第一个抛出的错误信息。
  
  `Promise.any()`
    只要其中的一个 promise 成功，就返回那个已经成功的 promise 。
    如果可迭代对象中没有一个 promise 成功（即所有的 promises 都失败/拒绝），就返回一个失败的 promise 和AggregateError类型的实例

  `Promise.race()`
    方法返回一个 promise，一旦迭代器中的某个promise解决或拒绝，返回的 promise就会解决或拒绝

new Promise发生了什么/过程

  Promise的构造函数接收一个参数，是函数，并且传入两个参数：resolve，reject，分别表示异步操作执行成功后的回调函数和异步操作执行失败后的回调函数

  ```js

    var p = new Promise(function(resolve, reject){
      //做一些异步操作
      setTimeout(function(){
        console.log('执行完成');
        resolve('随便什么数据');
      }, 2000);
    });
    // 上面的代码中，我们执行了一个异步操作，也就是setTimeout 2秒后，输出‘执行完了’，并且调用resolve方法

    // 运行代码，会在2秒后输出“执行完成”。注意！我只是new了一个对象，并没有调用它，我们传进去的函数就已经执行了，这是需要注意的一个细节。所以我们用Promise的时候一般是包在一个函数中，在需要的时候去运行这个函数

    function runAsync(){
      var p = new Promise(function(resolve, reject){
        //做一些异步操作
        setTimeout(function(){
            console.log('执行完成');
            resolve('随便什么数据');
        }, 2000);
      });
      return p;            
    }
    runAsync()
  ```

## Async/Await 如何通过同步的方式实现异步

Async/Await 是一个自执行的 generate 函数。利用 generate 函数的特性把异步的代码写成“同步”的形式

## JS 异步解决方案的发展历程以及优缺点

回调函数
  优点：解决了同步的问题（整体任务执行时长）；
  缺点：回调地狱，不能用 try catch 捕获错误，不能 return;

Promise
  优点：解决了回调地狱的问题；
  缺点：无法取消 Promise，错误需要通过回调函数来捕获；

Generator
  特点：可以控制函数的执行

Async/Await
  优点：代码清晰，不用像 Promise 写一大堆 then 链，处理了回调地狱的问题
  缺点：await 将异步代码改造成同步代码，如果多个异步操作没有依赖性而使用 await 会导致性能上的降低；

## 防抖和节流

防抖：触发高频事件后 n 秒内函数只会执行一次，如果 n 秒内高频事件再次被触发，则重新计算时间

```js
  function debounce(fn, timing) {
    let timer;
    return function() {
      clearTimeout(timer);
      timer = setTimeout(() => {
        fn();
      }, timing);
    }
  }
```

节流：高频事件触发，但在 n 秒内只会执行一次，所以节流会稀释函数的执行效率

```js
  function throttle(fn, timing) {
    let trigger;
    return function() {
      if (trigger) return;
      trigger = true;
      fn();
      setTimeout(() => {
        trigger = false;
      }, timing);
    }
  }
```

防抖靠定时器控制，节流靠变量控制

## Set、Map、WeakSet 和 WeakMap 的区别

Set

  1. 成员不能重复；
  2. 只有键值，没有键名，有点类似数组；
  3. 可以遍历，方法有 add、delete、has

WeakSet
  
  1. 成员都是对象（引用）；
  2. 成员都是弱引用，随时可以消失（不计入垃圾回收机制）。可以用来保存 DOM 节点，不容易造成内存泄露；
  3. 不能遍历，方法有 add、delete、has；

Map

  1. 本质上是键值对的集合，类似集合；
  2. 可以遍历，方法很多，可以跟各种数据格式转换；

WeakMap

  1. 只接收对象为键名（null 除外），不接受其他类型的值作为键名；
  2. 键名指向的对象，不计入垃圾回收机制；
  3. 不能遍历，方法同 get、set、has、delete；

## ES5/ES6 的继承除了写法以外还有什么区别

  1. class 声明会提升，但不会初始化赋值。（类似于 let、const 声明变量;
  2. class 声明内部会启用严格模式；
  3. class 的所有方法（包括静态方法和实例方法）都是不可枚举的
  4. class 的所有方法（包括静态方法和实例方法）都没有原型对象 prototype，所以也没有 `[[constructor]]`，不能使用 new 来调用
  5. 必须使用 new 来调用 class
  6. class 内部无法重写类名

## call 和 apply 的区别是什么，哪个性能更好一些

 1. Function.prototype.apply 和 Function.prototype.call 的作用是一样的，区别在于传入参数的不同
 2. 第一个参数都是指定函数体内 this 的指向
 3. 第二个参数开始不同，apply 是传入带下标的集合，数组或者类数组，apply 把它传给函数作为参数，call 从第二个开始传入的参数是不固定的，都会传给函数作为参数
 4. call 比 apply 的性能要好，call 传入参数的格式正式内部所需要的格式

## 实现 (5).add(3).minus(2) 功能

```js

Number.prototype.add = function(n) {
  return this + n;
}

Number.prototype.minus = function(n) {
  return this - n;
}
```

## ES6 代码转成 ES5 代码的实现思路是什么

Babel 的实现方式：

  1. 将代码字符串解析成抽象语法树，即所谓的 AST；
  2. 对 AST 进行处理，在这个阶段可以对 ES6 AST 进行相应转换，即转换成 ES5 AST；
  3. 根据处理后的 AST 再生成代码字符串；

## js 中有哪几种内存泄露的情况

  1. 意外的全局变量
  2. 闭包
  3. 未被清空的定时器
  4. 未被销毁的事件监听
  5. DOM 引用

## 懒加载

懒加载也叫延迟加载，指的是在长网页中延迟加载图像，是一种很好优化网页性能的方式。

懒加载的优点：

  1. 提升用户体验，加快首屏渲染速度；
  2. 减少无效资源的加载
  3. 防止并发加载的资源过多会阻塞 js 的加载；

懒加载的原理：

  首先将页面上的图片的 src 属性设为空字符串，而图片的真实路径则设置在 data-original 属性中，当页面滚动的时候需要去监听 scroll 事件，在 scroll 事件的回调中，判断我们的懒加载的图片是否进入可视区域，如果图片在可视区内则将图片的 src 属性设置为 data-original 的值，这样就可以实现延迟加载

## this

this 是和执行上下文绑定的

绑定规则
  
  1. 默认绑定
  2. 隐式绑定
  3. 显示绑定
  4. new绑定

优先级

  1. 函数是否在new中调用
  2. 函数是否通过call, apply (显示绑定) 或者硬绑定
  3. 函数是否在某个上下文对象中调用(隐式绑定)
  4. 如果不是 使用默认绑定 如果在严格模式下 就绑定到undefined 否则就绑定到全局对象

例外

  1. 如果你把 null / undefined 作为绑定的对象传入call / apply / bind , 这些值会在调用时忽略, 实际上应用的是默认绑定
  2. 箭头函数不使用this的4种标准, 而是根据外层(函数或全局)作用域来决定this (无法修改)

## new的过程

使用new来调用函数, 或者说发生构造函数调用时, 会自动执行下面的操作

1. 创建(或者说构建)一个全新的对象
2. 这个新对象会被执行 prototype 连接
3. 这个新对象会绑定到函数调用的this
4. 如果这个函数没有返回其它对象, 那么new表达式中的函数调用会自动返回这个对象

## 闭包

什么是闭包

  当函数可以记住并访问所在的词法作用域，即使函数在当前词法作用的域之外执行， 这时就产生了闭包

  闭包就是能够读取其他函数内部变量的函数

闭包的特征
  
  1. 函数内再嵌套函数
  2. 内部函数可以引用外层的参数和变量
  3. 参数和变量不会被垃圾回收制回收

对闭包的理解

  使用闭包主要是为了设计私有的方法和变量。闭包的优点是可以避免全局变量的污染，缺点是闭包会常驻内存，会增大内存使用量，使用不当很容易造成内存泄露

优/缺点

  优点：能够实现封装和缓存等

  缺点：就是消耗内存、不正当使用会造成内存溢出的问题

使用闭包的注意点

  由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露

  解决方法是：在退出函数之前，将不使用的局部变量全部删除

## 作用域/作用域链

作用域

  全局作用域：对象在代码中的任何地方都能访问，其生命周期伴随着页面的生命周期。

  函数作用域：函数内部定义的变量或者函数，并且定义的变量或者函数只能在函数内部被访问。函数执行结束之后，函数内部定义的变量会被销毁。

  局部作用域：使用一对大括号包裹的一段代码，比如函数、判断语句、循环语句，甚至单独的一个`{}`都可以被看作是一个块级作用域

作用域链

  一般情况下，变量取值到 创建 这个变量 的函数的作用域中取值。

  但是如果在当前作用域中没有查到值，就会向上级作用域去查，直到查到全局作用域，这么一个查找过程形成的链条就叫做作用域链

## 原型/原型链

概念

  每个对象都会在其内部初始化一个属性，就是prototype(原型)，当我们访问一个对象的属性时，如果这个对象内部不存在这个属性，那么他就会去prototype里找这个属性，这个prototype又会有自己的prototype，于是就这样一直找下去

原型和原型链的关系

  `instance.constructor.prototype = instance.__proto__`

原型和原型链的特点

  JavaScript对象是通过引用来传递的，我们创建的每个新对象实体中并没有一份属于自己的原型副本。当我们修改原型时，与之相关的对象也会继承这一改变
  当我们需要一个属性的时，Javascript引擎会先看当前对象中是否有这个属性， 如果没有的
  就会查找他的Prototype对象是否有这个属性，如此递推下去，一直检索到 Object 内建对象

属性设置和屏蔽:

  如果foo不直接存在于myObject中而是在于原型链上层时 myObject.foo = ‘bar‘ 会出现3种情况

  1. 如果原型链上层存在foo的普通数据访问属性,并且没有被标记为只读(writable: false), 那就会直接在myObject中添加一个名为foo的新属性, 它是一个屏蔽属性
  2. 如果原型链上层存在foo, 但是它被标记为只读(writable: false), 那么无法修改已有属性或在myObject上创建屏蔽属性. 如果是在严格模式下,代码会抛出一个错误. 否则, 这条赋值语句会被忽略. 总之, 不会发生屏蔽
  3. 如果原型链上层存在foo并且他是一个setter, 那就会调用这个setter, foo不会被添加到myObject, 也不会重新定义foo这个setter

  为什么会出现这种情况(屏蔽属性)
    主要是为了模拟类属性的继承

  这个限制只存在于 ‘=’赋值, 使用Object.defineProperty() 不受影响

## es6 的class语法糖 解决了什么问题

1. 更好看 (不再需要写 prototype)
2. 不再需要引用杂乱的‘.prototype’了
3. 不再需要通过Object.create() 来替换 ‘.prototype’对象 , 也不需要‘.__proto__’或者 Object.setProtoTypeOf
4. 可以通过super()  来实现相对多态, 这样任何方法都可以引用原型链上层的同名方法
5. class字面语法不能声明属性(只能声明方法), 看起来是一种限制, 但是它会排除掉许多不好的情况, 如果没有这种限制的话, 原型链末端的‘实例’可能会意外的获取其它地方的属性(这些属性隐式被所有‘实例’所‘共享’)
6. 可以通过extends很自然的扩展对象(子)类型, 比如Array或RegExp

## class 陷阱

1. class语法并不是向js中引入一种新的‘类’机制, 而是基于现有的 prototype机制的一种语法糖
a. 也就是说 es6 class并不会像传统面向类一样在声明时静态复制所有行为. 如果你修改或替换父类中的一个方法, 那子类和所有实例都会收到影响, 应为它们在定义时没有进行复制, 而是基于 prototype 的实例委托
2. class无法定义类成员属性
3. class语法可能会面临意外的屏蔽 例如 id属性屏蔽了id方法
4. super也会有一些细微的问题 super不是动态绑定的 它会在声明时 ‘静态’绑定

## 组件化

为什么要组件化开发

  有时候页面代码量太大，逻辑太多或者同一个功能组件在许多页面均有使用，维护起来相当复杂，这个时候，就需要组件化开发来进行功能拆分、组件封装，已达到组件通用性，增强代码可读性，维护成本也能大大降低

组件化开发的优点

  很大程度上降低系统各个功能的耦合性，并且提高了功能内部的聚合性。这对前端工程化及降低代码的维护来说，是有很大的好处的，耦合性的降低，提高了系统的伸展性，降低了开发的复杂度，提升开发效率，降低开发成本

组件化开发的原则

  1. 专一
  2. 可配置性
  3. 标准性
  4. 复用性
  5. 可维护性

## 模块化

为什么要模块化

  早期的javascript版本没有块级作用域、没有类、没有包、也没有模块，这样会带来一些问题，如复用、依赖、冲突、代码组织混乱等，随着前端的膨胀，模块化显得非常迫切

模块化的好处

  1. 避免变量污染，命名冲突
  2. 提高代码复用率
  3. 提高了可维护性
  4. 方便依赖关系管理

模块化的几种方法

  函数封装

  总结：这样避免了变量污染，只要保证模块名唯一即可，同时同一模块内的成员也有了关系
  缺陷：外部可以睡意修改内部成员，这样就会产生意外的安全问题

  立即执行函数表达式

  总结：这样在模块外部无法修改我们没有暴露出来的变量、函数
  缺点：功能相对较弱，封装过程增加了工作量，仍会导致命名空间污染可能、闭包是有成本的

## 数据类型检测的方式有哪些

### typeof

  ```js
    console.log(typeof 2);               // number
    console.log(typeof true);            // boolean
    console.log(typeof 'str');           // string
    console.log(typeof []);              // object    
    console.log(typeof function(){});    // function
    console.log(typeof {});              // object
    console.log(typeof undefined);       // undefined
    console.log(typeof null);            // object
  ```

  其中数组，对象，null都会被判断object 其它都正确

### instanceof

  ```js
    console.log(2 instanceof Number);                    // false
    console.log(true instanceof Boolean);                // false 
    console.log('str' instanceof String);                // false 
    
    console.log([] instanceof Array);                    // true
    console.log(function(){} instanceof Function);       // true
    console.log({} instanceof Object);                   // true
  ```

  `instanceof`运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上

   只能判断引用类型数据，而不能判断基本数据类型（字面量）

  ```js
    console.log(new Number(2) instanceof Number);                    // true
    console.log(new Boolean(true) instanceof Boolean);               // true 
    console.log(new String('str') instanceof String);                // true 
  ```

  字面值被实例化了，他们的判断值变为了 true

  特殊的两个值`null`和`undefined`
  
  ```js
    console.log(new null instanceof Null);  // null is not a constructor
    console.log(new undefined instanceof Undefined);  // undefined is not a constructor
  ```

  浏览器认为null，undefined不是构造器。但是在typeof中你可能已经发现了，`typeof null`的结果是object，`typeof undefined`的结果是undefined
  
  尤其是null，其实这是js发展过程中设计者的重大失误，早期准备更改null的类型为null，由于当时已经有大量网站使用了null，如果更改，将导致很多网站的逻辑出现漏洞问题，就没有更改过来，于是一直遗留到现在

### constructor

  ```js

  console.log((2).constructor === Number); // true
  console.log((true).constructor === Boolean); // true
  console.log(('str').constructor === String); // true
  console.log(([]).constructor === Array); // true
  console.log((function() {}).constructor === Function); // true
  console.log(({}).constructor === Object); // true

  ```

  这里`constructor`有两个作用

  1. 判断数据类型
  2. 对象实例通过constructor对象访问它的构造函数

  如果创建一个对象来改变它的原型，`constructor`就不能用来判断数据类型了

  ```js

    function Fn(){};
    Fn.prototype = new Array();
    var f = new Fn();
    
    console.log(f.constructor===Fn);    // false
    console.log(f.constructor===Array); // true

  ```
  
### Object.prototype.toString.call

  使用object对象原型方法toString 判断类型

  ```js
    const a = Object.prototype.toString;
    
    console.log(a.call(2));             // [object Number]
    console.log(a.call(true));          // [object Boolean]
    console.log(a.call('str'));         // [object String]
    console.log(a.call([]));            // [object Array]
    console.log(a.call(function(){}));  // [object Function]
    console.log(a.call({}));            // [object Undefined]
    console.log(a.call(undefined));     // [object Undefined]
    console.log(a.call(null));          // [object Null]
  ```

  同样是检测对象obj调用toString方法，`obj.toString()`的结果和`Object.prototype.toString.call(obj)`的结果不一样，这是为什么?

  这是因toString为Object的原型方法，而Array。function等类型作为object的实例，都重写了toString方法。不同对象类型调用toString方法时会根据原型，调用的是对应重写之后的toString方法（function类型返回内容为函数体的字符串，Array类型返回元素组成的字符串...），而不会调用Object原型上的toString方法（返回对象的具体类型），所以`obj.toString()`不能得到其对象类型，只能将obj转换为字符串类型；因此，如果想要得到对象的具体类型时，应该调用Object原型上的toString方法

## 为什么0.1+0.2 ! == 0.3，如何让其相等

对于这个问题直接的解决方法就是设置一个误差范围，通常称为`机器精度`。

在es6中，提供了`Number.EPSILON`属性，只要判断`0.1 + 0.2 - 0.3`是否小于`Number.EPSILON`，如果小于，就可以判断`0.1+0.2 === 0.3`

```js

function numberepsilon(arg1,arg2){                   
  return Math.abs(arg1 - arg2) < Number.EPSILON;        
}        
console.log(numberepsilon(0.1 + 0.2, 0.3)); // true

```
