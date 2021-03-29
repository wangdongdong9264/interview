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

todo

Pick

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
