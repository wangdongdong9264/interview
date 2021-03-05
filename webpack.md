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
