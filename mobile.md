# 移动端

## 移动端点击

移动端 300 ms 点击（click 事件）延迟
  由于移动端会有双击缩放的这个操作，因此浏览器在 click 之后要等待 300ms，判断这次操作是不是双击

  解决方案：

  1. 禁用缩放：user-scalable=no
  2. 更改默认的视口宽度
  3. CSS touch-action

## 点击穿透问题

  因为 click 事件的 300ms 延迟问题，所以有可能会在某些情况触发多次事件

  解决方案：
    1. 只用 touch
    2. 只用 click

## 移动端适配

1. 写页面时，按照设计稿写固定宽度，最后再统一缩放处理，在不同手机上都能用
2. 按照设计稿的标准开发页面，在手机上部分内容根据屏幕宽度等比缩放，部分内容按需要变化，需要缩放的元素使用 rem, vw 相对单位，不需要缩放的使用 px
3. 固定尺寸+弹性布局，不需要缩放

viewport适配

  ```html
  <meta name="viewport" content="width=750,initial-scale=0.5">
  <!-- initial-scale = 屏幕的宽度 / 设计稿的宽度 -->

  <head>
    <script>
      const WIDTH = 750
      const mobileAdapter = () => {
        let scale = screen.width / WIDTH
        let content = `width=${WIDTH}, initial-scale=${scale}, maximum-scale=${scale}, minimum-scale=${scale}`
        let meta = document.querySelector('meta[name=viewport]')
        if (!meta) {
          meta = document.createElement('meta')
          meta.setAttribute('name', 'viewport')
          document.head.appendChild(meta)
        }
        meta.setAttribute('content',content)
      }
      mobileAdapter()
      window.onorientationchange = mobileAdapter //屏幕翻转时再次执行
    </script>
  </head>

  ```

  缺点： 就是边线问题，不同尺寸下，边线的粗细是不一样的（等比缩放后），全部元素都是等比缩放，实际显示效果可能不太好

vw适配（部分等比缩放）：

  1. 开发者拿到设计稿（假设设计稿尺寸为750px，设计稿的元素标注是基于此宽度标注）
  2. 开始开发，对设计稿的标注进行转换，把px换成vw。比如页面元素字体标注的大小是32px，换成vw为 (100/750)*32 vw
  3. 对于需要等比缩放的元素，CSS使用转换后的单位
  4. 对于不需要缩放的元素，比如边框阴影，使用固定单位px

  关于换算，为了开发方便，利用自定义属性，CSS变量

  ```html
  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1">
    <script>
      const WIDTH = 750
      //:root { --width: 0.133333 } 1像素等于多少 vw
      document.documentElement.style.setProperty('--width', (100 / WIDTH)) 
    </script>
  </head>

  ```

  注意此时，meta 里就不要去设置缩放了

  ```css
  header {
    font-size: calc(28vw * var(--width))
  }

  ```

## 移动端1px

利用 css 的 伪元素::after + transfrom 进行缩放

  为什么用伪元素

  因为伪元素::after或::before是独立于当前元素，可以单独对其缩放而不影响元素本身的缩放

  伪元素大多数浏览器默认单引号也可以使用，和伪类一样形式，而且单引号兼容性（ie）更好些

  ```css

  div::after{
    content:'';width:100%;
    border-bottom:1px solid #000;
    transform: scaleY(0.5);
  }
  ```

其它方案

  使用图片：兼容性最好，灵活行最差，不能改变颜色、长度

  使用 viewport 和 rem，js 动态改变 viewport 中 scale 缩放，缺点在于不适用于已有的项目
  例如：使用 vh 和 vw 布局的
  
  ```html

  <meta name="viewport" id="WebViewport" content="initial-scale=1,    maximum-scale=1, minimum-scale=1, user-scalable=no">

  ```

  使用 css 渐变linear-gradient或者box-shadow

## 骨架屏原理

  骨架屏是在内容还没有出现之前的页面骨架填充，以免留白

原理：

  因为render生成的vNode，通过`$mount`方法，挂载在我们的定义的 DOM 元素上；这里的挂载是`替换`的意思。

实现方案：

1. 在`index.html`中的`div#app`中来实现骨架屏，程序渲染后就会替换掉index.html里面的`div#app`骨架屏内容
2. 使用一个Base64的图片来作为骨架屏
3. 在构建时使用 Vue 预渲染功能，将骨架屏组件的渲染结果 HTML 片段插入 HTML 页面模版的挂载点中，将样式内联到 head 标签中`vue-skeleton-webpack-plugin`
