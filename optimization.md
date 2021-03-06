# 优化

## 你有对 Vue 项目进行哪些优化

代码层面的优化

  1. v-if 和 v-show 区分使用场景
  2. computed 和 watch  区分使用场景
  3. v-for 遍历必须为 item 添加 key，且避免同时使用 v-if
  4. 长列表性能优化
  5. 事件的销毁
  6. 图片资源懒加载
  7. 路由懒加载
  8. 第三方插件的按需引入
  9. 优化无限列表性能
  10. 服务端渲染 SSR or 预渲染

Webpack 层面的优化

  1. Webpack 对图片进行压缩
  2. 减少 ES6 转为 ES5 的冗余代码
  3. 提取公共代码
  4. 模板预编译
  5. 提取组件的 CSS
  6. 优化 SourceMap
  7. 构建结果输出分析
  8. Vue 项目的编译优化

基础的 Web 技术的优化

  1. 开启 gzip 压缩
  2. 浏览器缓存
  3. CDN 的使用
  4. 使用 Chrome Performance 查找性能瓶颈

## vue大数据量场景的性能优化

原因:

  Vue在渲染组件的时候，会对data对象进行改造，遍历data的key，调用defineProperty方法定义它的setter和getter。如果某个字段是Object，或者Array，还会递归的对这个字段进行上诉操作

  通常情况下，这个操作耗时是很短的，但是当数据量非常大的时候，对每一个数据项的每一个字段都进行defineProperty的操作就是一个昂贵的操作，所以性能优化的出发点就是减少defineProperty的次数

优化方案：

* 减少无用字段

    在实际项目中，其实我只需要2个字段，一个是name，一个是id（id甚至也可以不要），所以我把多余的字段都去掉，一共减少了8个String类型的字段，和一个Object类型的字段，可以减少 `(8 + 4) * n`次defineProperty操作和n次递归调用

* 数据扁平化

    在当前版本（2.x）的Vue当中，对于数据变动的检测有许多限制，比如不能检测对象属性的添加和删除；不能检测到通过数据索引直接设置数据项等等。所以，当一个数组的数据项都是基本数据类型的时候，Vue不会进行任何操作

* 利用computed

    性能上的问题已经解决了，但是扁平的数据会影响业务代码的开发效率和可读性，同时数据和它的index产生了深耦合，如果我们需要添加一个字段使用或者改变下顺序，很容易出问题。不过，我们可以利用computed计算属性把已经被拍扁的数据重新组装起来。由于Vue的响应式数据改造只针对data选项和props选项，不包括computed，所以只会产生很少的函数运行耗时

* 数据静态化

    常在Vue组件当中，都是把数据放在data选项当中，Vue会对data选项中的数据进行响应式改造，我称之"动态数据"或者"响应式"数据，但是并不是所有的数据都会发生变化的，很多时候，特别是大数据量场景下的数据是不会或者很少发生变化的，这种情况下，就没有必要把它放到data选项中去，在beforeCreated当中进行数据初始化，也不会影响数据的使用

    静态数据并不在Vue的响应式系统当中，也就是说当你进行this.userList = newUserList时，视图不会重新渲染，对应的computed计算属性也不会重新计算。没有了Vue提供的响应式系统，如果数据变动的时候，我们需要手动的去计算对应的数据，可能还需要配合$forceUpdate这个api去重新渲染视图。此时，需要在性能和代码可读性与开发效率上进行取舍与权衡
