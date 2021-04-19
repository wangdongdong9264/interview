# typescript

## ts的基本数据类型

数字类型： number： 双精度 64 位浮点值。它可以用来表示整数和分数

字符串类型： string： 字符串

布尔类型： boolean： 表示逻辑值：true 和 false

null：null： 表示对象值缺失

undefined： undefined：用于初始化变量为一个未定义的值

任意类型： any ： 声明为 any 的变量可以赋予任意类型的值

数组类型： 无： 声明变量为数组

```ts
  // 在元素类型后面加上[]
  let arr: number[] = [1, 2];
  // 或者使用数组泛型
  let arr: Array<number> = [1, 2];
```

object: object : object表示非原始类型，也就是除number，string，boolean，symbol，null或undefined之外的类型

元组： 无： 元组类型用来表示已知元素数量和类型的数组，各元素的类型不必相同，对应位置的类型需要相同

```ts
let x: [string, number];
x = ['Runoob', 1];    // 运行正常
x = [1, 'Runoob'];    // 报错
console.log(x[0]);    // 输出 Runoob
```

枚举： enum： 枚举类型用于定义数值集合

```ts
enum Color {Red, Green, Blue};
let c: Color = Color.Blue;
console.log(c);    // 输出 2
```

void： void：用于标识方法返回值的类型，表示该方法没有返回值

unknown : unknown : 任何类型都可以赋值给unkown类型，但是unkown类型不可以赋值给除any以外的类型

类型断言： as ： 当你清楚地知道一个变量具有比它现有类型更确切的类型，对当前类型进行类型转换

```ts
//类型转换方式有两种
let str: any = "this is string"；
let strlength1: number = (str as sting).length; //建议使用这种方式
let strlength2: number = (<string>str).length;  //有兼容性问题
```

## interface和type的区别

相同点：都可以拓展（extends）

不同点：
  type 可以声明基本类型别名，联合类型，元组等类型

  type 语句中还可以使用 `typeof` 获取实例的类型进行赋值

  ```ts

  let div = document.createElement('div')
  type B = typeof div

  ```

  interface 能够声明合并

  ```ts
  interface User {
    name: string
    age: number
  }
  interface User {
    sex: string
  }

  // User 接口为 {
  //   name: string
  //   age: number
  //   sex: string
  // }
  ```

## never和void的区别

  void 表示没有任何类型（可以被赋值为 null 和 undefined）

  never 表示一个不包含值的类型，即表示永远不存在的值

  拥有 void 返回值类型的函数能正常运行。拥有 never 返回值类型的函数无法正常返回，无法终止，或会抛出异常

## 什么是泛型

  泛型是指在定义函数、接口或类的时候，不预先指定具体的类型，使用时再去指定类型的一种特性

  可以把泛型理解为代表类型的参数

## 一些常用的工具泛型

需要了解的关键字

`keyof`可以用来回取得一个对象接口的所有key值

```ts

interface Foo {
  name: string;
  age: number
}
type T = keyof Foo // -> "name" | "age"

```

`in`可以遍历枚举类型

```ts

ype Keys = "a" | "b"
type Obj =  {
  [p in Keys]: any
} // -> { a: any, b: any }

```

`extends`用来约束类型

```ts

K extends keyof T
// 应该是说K被约束在T的key中，不能超出这个范围

```

`infer`在条件类型语句中, 我们可以用 infer 声明一个类型变量并且对它进行使用

### Partial

  作用是将传入的属性变为可选项
  
  ```ts

  type Partial<T> = { [p in keyof T]?: T[p] }

  ```

### Required

  作用是将传入的属性变为必选项

  ```ts
  type Required<T> = { [p in keyof T]-?:T[p] }

  ```

  `-?` 这里很好理解就是将可选项代表的 ? 去掉, 从而让这个类型变成必选项. 与之对应的还有个`+?` , 这个含义自然与`-?`之前相反, 它是用来把属性变成可选项的

### Readonly

  将传入的属性变为只读选项

  ```ts

  type Readonly<T> = { readonly [ p in keyof T]: T[p] }

  ```

### Record

  将 K 中所有的属性的值转化为 T 类型

  ```ts

  type Record<k extends keyof any, T> = { [p in k]: T}

  // 例子
  type petsGroup = 'dog' | 'cat' | 'fish';
  interface IPetInfo {
      name:string,
      age:number,
  }
  type IPets = Record<petsGroup, IPetInfo>;
  const animalsInfo:IPets = {
      dog:{
          name:'dogName',
          age:2
      },
      cat:{
          name:'catName',
          age:3
      },
      fish:{
          name:'fishName',
          age:5
      }
  }

  ```

### Pick

  从 T 中取出 一系列 K 的属性

  ```ts

  type Pick<T, K extends keyof T> = { [P in K]: T[P] }

  // 原始类型
  interface TState {
    name: string;
    age: number;
    like: string[];
  }
  // 如果我只想要name和age怎么办，最粗暴的就是直接再定义一个（我之前就是这么搞得）
  interface TSingleState {
    name: string;
    age: number;
  }
  // 这样的弊端是什么？就是在Tstate发生改变的时候，TSingleState并不会跟着一起改变，所以应该这么写
  interface TSingleState extends Pick<TState, "name" | "age"> {};

  ```

### Exclude

  从 T 中找出 U 中没有的元素, 换种更加贴近语义的说法其实就是从T 中排除 U

  对于联合类型来说会自动分发条件，例如 `T extends U ? X : Y, T` 可能是 A | B 的联合类型, 那实际情况就变成`(A extends U ? X : Y) | (B extends U ? X : Y)`

  ```ts

  type Exclude<T, U> = T extends U ? never : T

  // 例子
  type T = Exclude<1 | 2, 1 | 3> // 2  

  ```

### Extract

  作用是提取出 T 包含在 U 中的元素, 换种更加贴近语义的说法就是从 T 中提取出 U

  ```ts

  type Extract<T, U> = T extends U ? T : never

  ```

### 组合形式

  Pick 和 Exclude 进行组合, 实现忽略对象某些属性功能

  ```ts

  type Omit<T, K> = Pick<T, Exclude<keyof T, K>>

  // 例子
  type Foo = Omit<{name: string, age: number}, 'name'> // { age: number }

  ```

### ReturnType

  用它获取函数的返回类型

```ts

type ReturnType<T> = T extends (...args: any[]) => infer P ? P : any;

// 例子
function foo(x: number): Array<number> {
  return [x];
}
type fn = ReturnType<typeof foo>;

```

## 用过 TypeScript 吗？它的作用是什么？

  TS 在开发时就能给出编译错误， 而 JS 错误则需要在运行时才能暴露

  作为强类型语言，你可以明确知道数据的类型。代码可读性极强，几乎每个人都能理解

  为 JS 添加类型支持，以及提供最新版的 ES 语法的支持，是的利于团队协作和排错，开发大型项目

## extends 和 implement 的区别

implements关键字将类A当作一个接口，这意味着类C必须去实现定义在A中的所有方法，无论这些方法是否在类A中有没有默认的实现。同时，也不用在类C中定义super方法

extends关键字本身所表达的意思一样，你只需要实现类A中定义的虚方法，并且关于super的调用也会有效

## 什么是可索引类型接口

  一般用来约束数组和对象

  ```ts
  // 数字索引——约束数组
  // index 是随便取的名字，可以任意取名
  // 只要 index 的类型是 number，那么值的类型必须是 string
  interface StringArray {
    // key 的类型为 number ，一般都代表是数组
    // 限制 value 的类型为 string
    [index:number]:string
  }
  let arr:StringArray = ['aaa','bbb'];
  console.log(arr);


  // 字符串索引——约束对象
  // 只要 index 的类型是 string，那么值的类型必须是 string
  interface StringObject {
    // key 的类型为 string ，一般都代表是对象
    // 限制 value 的类型为 string
    [index:string]:string
  }
  let obj:StringObject = {name:'ccc'};
  ```

## 装饰器到注解到元编程

`装饰器`（Decorator）仅提供定义和劫持，能够对类及方法，方法入参，属性的定义并没有提供任何附加元数据的功能。

`注解`（Annotation）仅提供附加元数据支持，并不能实现任何操作。需要另外的`Scanner`根据元数据执行相应操作

`元编程` 类及其实例并不能感知或者修改存取在类上元数据，但是我们可以通过`装饰器`和`注解`在编译时动态的修改它们的行为，即我们写了一个函数去修改函数，这样的行为称作元编程

其它：

  装饰器是在编译期间发生的，这个时候类的实例还没有生成，因此装饰器无法直接对类的实例进行修改。但是可以间接的通过修改类的原型影响实例

  要进行元数据的修改，我们需要利用反射`Reflect`。 ES6提供的Refelct并不满足修改元数据，我们要额外引入一个库`reflect-metadata`

  反射`Reflect`给了我们在类及其属性、方法、入参上存储读取数据的能力
