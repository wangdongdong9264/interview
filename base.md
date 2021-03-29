# 基础

## 数组遍历方法总结

### for循环

  没啥说的，`循环条件`总比`循环体`多执行一次

### for...in循环

  不建议用数组上，`for...in`是为了遍历对象属性而构建的

  `for…in`语句以任意顺序遍历一个对象的除Symbol以外的可枚举属性

  如果通过`arr.name = '李明'`给数组添加一个属性是可枚举的，甚至可以遍历到你给`Array.prototype`新增加的属性与方法

### for...of循环

  `for…of`语句在 可迭代对象（包括 Array，Map，Set，String，TypedArray，arguments对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句

  es6新增的方法，针对`for...in`循环的bug，`for...of`就不会有

  使用`for...of` 循环遍历数组，其结果就是打印出数组中的每一个值

  如果使用for…of 循环时也想得到数组的下标或者是key，可以利用`arr.keys()`

  ```js
  const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
  for (let key of arr.keys()) {
      console.log(key)
  }
  //  输出结果：
  //  0 1 2 3 4 5 6 7 8
  ```

  如果使用for…of循环，既想得到key 也想得到 value ，可以利用`arr.entries()`

  ```js
  const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
  for (let [key, val] of arr.entries()) {
      console.log(key, val)
  }
  //  输出结果：
  //  0,  1 
  //  1,  2 
  //  2,  3 
  //  3,  4 
  //  4,  5 
  //  5,  6 
  //  6,  7 
  //  7,  8 
  //  8,  9 
  ```

### forEach循环

  用于调用数组的每个元素，并将元素传递给回调函数

  对于空数组是不会执行回调函数的

  接受三个参数，
    第一参数为数组中的每一项
    第二参数为数组的下标
    第三个参数为你要遍历的数组本身 第二和第三参数都是可选的

  本身不支持 continue 和 break语句的
    如果想实现continue效果，可以使用 return
    如果要实现break效果，建议使用 `every` 和 `some` 循环

### map循环

  返回一个经过调用函数处理后的新的数组

  不会对空数组进行检测

  必须 return

  接受三个参数（同forEach）

  会针对每一项都进行循环，如果跳过则会返回 undefined

  ```js

  const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
  const res = arr.map(item=>{
      if(item > 3) {
          return item;
      }
  })
  console.log(res)

  // 输出结果：
  // [undefined, undefined, undefined, 4, 5, 6, 7, 8, 9] 

  ```

### filter循环

  返回一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素

  不会对空数组进行检测

  不会改变原数组
  
  接受三个参数（同forEach）

### some循环

  查找数组中任意符合条件的元素并返回boolean值，当数组中有任意元素符合条件就返回 true 否则返回 fasle

  会依次执行数组的每一个元素

  如果有一个元素满足条件，则返回 true，且剩余的元素不会在执行检测 即 循环结束

  不会对空数组进行检测

  接受三个参数（同forEach）

### every循环

  每一个元素都满足条件 则返回 true，否则返回 false（与some相反）

### find

  历数组，返回符合条件的第一个元素，如果没有符合条件的元素则返回 undefined

### findIndex

  遍历数组，返回符合条件的第一个元素的索引，如果没有符合条件的元素则返回 -1

### reduce 循环

  接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值

  接收四个参数
    第一参数为数组中的每一项累加和
    第二参数为数组的每一项
    第三个参数为数组的下标
    第四个参数为你要遍历的数组本身。第三和第四参数都是可选的

## 对象遍历的方法总结

### for in

  最基础的遍历对象的方式，它还会得到对象原型链上的属性

  ```js
  // 以使用对象的 hasOwnProperty() 方法过滤掉原型链上的属性
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      console.log(obj[key]) // foo
    }
  }

  ```

### Object.keys

  `Object.keys()` 方法会返回一个由一个给定对象的自身可枚举属性组成的数组，会自动过滤掉原型链上的属性

  ```js

  Object.keys(obj).forEach((key) => {
    console.log(obj[key]) // foo
  })

  ```

  `Object.values()`方法返回一个给定对象自身的所有可枚举属性值的数组

  `Object.entries()`方法返回一个给定对象自身可枚举属性的键值对数组

  ```js

  const obj = { foo: 'bar', baz: 42 };
  console.log(Object.entries(obj)); // [ ['foo', 'bar'], ['baz', 42] ]

  for (const [key, value] of Object.entries(obj)) {
    console.log(`${key}: ${value}`);
  }

  ```

### Object.getOwnPropertyNames

  `Object.getOwnPropertyNames()`方法返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组

### Object.getOwnPropertySymbols

  `Object.getOwnPropertySymbols()` 方法返回一个给定对象自身的所有 Symbol 属性的数组

  ```js

  // 给对象添加一个不可枚举的 Symbol 属性
  Object.defineProperties(obj, {
   [Symbol('baz')]: {
    value: 'Symbol baz',
    enumerable: false
   }
  })
   
  // 给对象添加一个可枚举的 Symbol 属性
  obj[Symbol('foo')] = 'Symbol foo'
   
  Object.getOwnPropertySymbols(obj).forEach((key) => {
   console.log(obj[key]) // Symbol baz, Symbol foo
  })

  ```

### Reflect.ownKeys

  [关于 Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

  `Reflect.ownKeys()`静态方法返回一个由目标对象自身的属性键组成的数组

  ```js
  const object1 = {
    property1: 42,
    property2: 13
  };

  const array1 = [];

  console.log(Reflect.ownKeys(object1));
  // expected output: Array ["property1", "property2"]

  console.log(Reflect.ownKeys(array1));
  // expected output: Array ["length"]
  ```

| 遍历方式/支持特性      | 遍历sting属性 | 遍历Symbol属性  | 遍历不可枚举    | 遍历原型链    |
| :---        |    :----:   |    :----:   |    :----:   |    :----:   |
| `for...in`      | true       | false   | false     | true     |
| `Object.key()`   | true        | false      | false     | false     |
| `Object.getOwnPropertyNames()`   | true        | false      | true     | false     |
| `Object.getOwnPropertySymbols()`   | false        | true      | true     | false     |
| `Reflect.ownKeys()`   | true        | true      | true     | false     |
