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
