# webpack

## 介绍下 webpack 热更新原理，是如何做到在不刷新浏览器的前提下更新页面的

  1. 当修改了一个或多个文件；
  2. 文件系统接收更改并通知 webpack；
  3. webpack 重新编译构建一个或多个模块，并通知 HMR（Hot Module Replacement） 服务器进行更新；
  4. HMR Server 使用 Websocket 通知 HMR runtime 需要更新，HMR runtime 通过 HTTP 请求更新 jsonp；
  5. HMR runtime 替换更新中的模块，如果确定这些模块无法更新，则触发整个页面刷新；

## webpack 中 loader 和 plugin 的区别是什么

  loader：loader 是一个转换器，将 A 文件进行编译成 B 文件，属于单纯的文件转换过程
  plugin：plugin 是一个扩展器，它丰富了 webpack 本身，针对是 loader 结束后，webpack 打包的整个过程，它并不直接操作文件，而是基于事件机制工作，会监听 webpack 打包过程中的某些节点，执行广泛的任务。

## webpack4 构建速度/体积优化

### 使用高版本的webpack和node

例如webpack3 升级 webpck4

webpack4的原因:
  
  1. V8带来的优化（for of 代替了forEach、Map和set代替了Object、includes代替indexOf）
  2. 默认使用更快的md4 hash算法 去替代 MD5
  3. webpacks AST 可以直接从loader传递给AST, 减少解析时间
  4. 使用字符串方法替代正则表达式

### 多进程，多实例 解析构建

方案：

  1. thread-loader
  2. parallel-webpack
  3. HappyPack

HappyPack去解析资源原理：

  每次webpack解析一个模块，HappyPack会将它及它的依赖分配给worker线程中

### 多进程，多实例：并行压缩

1. parallel-uglify-plugin
2. terser-webpack-plugin 开启parallel参数

```js
// terser-webpack-plugin
module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        parallel: true, // parallel默认值是当前电脑环境CPU数量的2倍减1
      }),
    ],
  },
};
```

webpack v5 自带最新的 terser-webpack-plugin

### 预编译资源模块

1. html-webpack-externals-plugin （例如将react，react-dom基础包通过cdn引入，不打入bundle中）
2. SplitChunksPlugin
3. 使用DLLPlugin进行分包，DIIReferencePlugin对mainfest.json引用 （官方内置的插件）

dllPlugin的使用：
一般先创建webpack.vendor.config.js文件对公共基础包、业务基础包进行分包

```js
const path = require('path');

new webpack.DllPlugin({
  context: __dirname,
  name: '[name]_[fullhash]',
  path: path.join(__dirname, 'manifest.json'),
});

```

然后在webpack.config.js通过webpack.DllReferenctPlugin进行引用

```js
new webpack.DllReferencePlugin({
  context: __dirname,
  manifest: require('./manifest.json')
});
```

### 利用缓存提升二次构建速度

1. babel-loader 开启缓存
2. terser-webpack-plugin开启缓存
3. 使用 cache-loader 或者 hard-source-webpack-plugin

### 缩小构建目标

目的：尽可能的减少构建模块

例如babel-loader不解析node_modules

减少文件搜索范围

1. 优化resolve.modules 配置 （减少模块搜索层级）
2. 优化resolve.mainFields配置
3. 优化resolve.extensions配置
4. 合理使用 alias

### 使用Tree Shaking 擦除无用的JavaScript和CSS

关于 tree shaking

  概念: 1个模块可能有多个方法，只要其中的某个方法使用到了，则整个文件都会被打到bundle里面去，tree shaking就是只把用到的方法打入bundle，没用到的方法会在uglify阶段被擦除掉。

  使用：webpack默认支持，在.babelrc里面设置modules：false即可
  production mode 的情况下默认开启

  要求：必须是ES6语法，CJS的方法不支持

擦除无用的CSS

  使用`purgecss-webpack-plugin`和`mini-css-extract-plugin` 擦除无用的CSS

### 使用 polyfill 动态服务

动态 polyfill 指的是根据不同的浏览器，动态载入需要的 polyfill。 Polyfill.io 通过尝试使用 polyfill 重新创建缺少的功能，可以更轻松地支持不同的浏览器，并且可以大幅度的减少构建体积

原理：识别 User Agent，下发不同的 Polyfill

使用方法：在 index.html 中引入如下 script 标签

```html

<script crossorigin="anonymous" src="https://polyfill.io/v3/polyfill.min.js"></script>

```

### 使用webpack进行图片压缩

使用`image-webpack-loader`

## dllPlugin 原理分析

`.dll`为后缀的文件称为动态链接库，在一个动态链接库中可以包含给其他模块调用的函数和数据
把基础模块独立打包出来放到单独的动态链接库里
当需要导入的模块在动态链接库的时候，模块不能再次打包，而且从动态链接库里面获取

DllPlugin插件：用于打包一个个动态链接库
DllReferencePlugin:在配置文件中引入DllPlugin插件打包好的动态链接库

一般放在生产配置就好了 开发环境建议不要用 比如vue库的一些错误提示会打包成时被省略了 ，会开发环境不友好

原理：

xx.manifest.json
name指的是对应的dll库名字
描述了哪些模块被打进来dll了， 用模块名当id标识出来 大概是一个模块清单

xxx.dll.js 就是模块的源码了
xxx.dll.js 是各个模块的源码集合 通过key（模块id）–> value查询出来

### webpack如何查找打包好的模块呢？而不需要重复打包

比如index.js引入了 react

```js
import React from 'react';
//./node_modules/_react@16.13.1@react/index.js

console.log(React);

```

webpack会去mainfest.json找模块id 其实就是react关键字拼接版本号 + index.js组成id去寻找模块
即 './node_modules/' + '_react@16.13.1@react' + '/index.js' 组成的id

找到有模块的话，就不再去打包了
打包出来的bundle.js直接会引入上面打包好的dll库，dll库会暴露一个全局变量xxx

bundle.js里面 会有一个定义好的key `dll-reference xxx` 对应 打包出来的全局变量，对应是全部的打包出来的第三方模块

index.js实际引入了react
bundle.js里面就有
`"./src/index.js": webpack_require(/*! react */ "./node_modules/_react@16.13.1@react/index.js");`

而 `webpack_require(/*! react */ "./node_modules/_react@16.13.1@react/index.js");` 其实就是指向刚才的全局变量

"./node_modules/_react@16.13.1@react/index.js"就是"dll-reference xxx"里面的某个key读取
所以可以通过`(webpack_require(/*! dll-reference xxx */ "dll-reference _dll_react"))("./node_modules/_react@16.13.1@react/index.js")`就能读取了react代码了
