# 手写代码

## new

```js
function myNew(fn) {
  const newObj = Object.create(fn.prototype);
  result = fn.apply(newObj, [...arguments].slice(1));
  return typeof result === "object" ? result : newObj;
}
```

## call

```js
Function.prototype.MyCall = function (context) {
  const args = [...arguments].slice(1);

  context.fn = this;

  const result = context.fn(...args);
  delete context.fn;

  return result;
}

```

## apply

```js
Function.prototype.MyApply = function (context) {
  const args = arguments[1] || [];

  context.fn = this;
  const result = context.fn(...args);
  delete context.fn;

  return result;
}

```

## bind

```js
// Function.prototype.MyBind = function (context) {
//   const args = [...arguments].slice(1);

//   return function () {
//     context.MyApply(context, args);
//   }
// }

// 考虑对原型链的影响
Function.prototype.bind2 = function(content) {
    if(typeof this != "function") {
        throw Error("not a function")
    }
    // 若没问参数类型则从这开始写
    let fn = this;
    let args = [...arguments].slice(1);
    
    let resFn = function() {
        return fn.apply(this instanceof resFn ? this : content,args.concat(...arguments) )
    }
    function tmp() {}
    tmp.prototype = this.prototype;
    resFn.prototype = new tmp();
    
    return resFn;
}

```

## instanceof

```js
function _instanceof(L, R) { //L为instanceof左表达式，R为右表达式
  let Ro = R.prototype //原型
  L = L.__proto__ //隐式原型
  while (true) {
    if (L === null) { //当到达L原型链顶端还未匹配，返回false
      return false
    }
    if (L === Ro) { //全等时，返回true
      return true
    }
    L = L.__proto__
  }
}
```

## promise

```js

const PENDING = "pending";
const RESOLVED = "resolved";
const REJECTED = "rejected";

class MyPromise {
  constructor(fn) {
    this.state = PENDING;
    this.resolvedHandlers = [];
    this.rejectedHandlers = [];
    fn(this.resolve.bind(this), this.reject.bind(this));
    return this;
  }

  resolve(props) {
    setTimeout(() => {
      this.state = RESOLVED;
      const resolveHandler = this.resolvedHandlers.shift();
      if (!resolveHandler) return;

      const result = resolveHandler(props);
      if (result && result instanceof MyPromise) {
        result.then(...this.resolvedHandlers);
      }
    });
  }

  reject(error) {
    setTimeout(() => {
      this.state = REJECTED;
      const rejectHandler = this.rejectedHandlers.shift();
      if (!rejectHandler) return;

      const result = rejectHandler(error);
      if (result && result instanceof MyPromise) {
        result.catch(...this.rejectedHandlers);
      }
    });
  }

  then(...handlers) {
    this.resolvedHandlers = [...this.resolvedHandlers, ...handlers];
    return this;
  }

  catch(...handlers) {
    this.rejectedHandlers = [...this.rejectedHandlers, ...handlers];
    return this;
  }
}

MyPromise.all = function (promises) {
  return new MyPromise((resolve, reject) => {
    const results = [];
    for (let i = 0; i < promises.length; i++) {
      const promise = promises[i];
      promise.then(res => {
        results.push(res);
        if (results.length === promises.length) resolve(results);
      }).catch(reject);
    }
  });
}

MyPromise.race = function (promises) {
  return new MyPromise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      const promise = promises[i];
      promise.then(resolve).catch(reject);
    }
  });
}

```

## 数组扁平化

```js

function flatten(arr) {
  let result = [];

  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      result = result.concat(flatten(arr[i]));
    } else {
      result = result.concat(arr[i]);
    }
  }

  return result;
}

const a = [1, [2, [3, 4]]];
console.log(flatten(a));

```

## 实现柯里化

预先设置一些参数

柯里化是什么：是指这样一个函数，它接收函数 A，并且能返回一个新的函数，这个新的函数能够处理函数 A 的剩余参数

```js

function createCurry(func, args) {
  var argity = func.length;
  var args = args || [];
  
  return function () {
    var _args = [].slice.apply(arguments);
    args.push(..._args);
    
    if (args.length < argity) {
      return createCurry.call(this, func, args);
    }
    
    return func.apply(this, args);
  }
}


```
