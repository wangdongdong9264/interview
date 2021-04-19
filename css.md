# css

## css盒模型

  box-sizing：content-box   //标准盒模型 width = content
  box-sizing：border-box    //怪异盒模型 width = content + padding + border

## 边距折叠

  块级元素的上外边距和下外边距有时会合并（或折叠）为一个外边距，其大小取其中的最大者，这种行为称为外边距折叠（margin collapsing），有时也翻译为外边距合并。

  注意浮动元素和绝对定位元素的外边距不会折叠。

  计算原则

  1. 两个都为正值直接去最大值;
  2. 两个一正一副时, 使用正值去减去负值的绝对值;
  3. 两个都为负值时, 两个都使用绝对值, 在使用0减去最大值。

  解决办法

    1. 兄弟间重叠时
       1. 底部元素变为行内盒子(display: inline-block);
       2. 底部元素设置flot
       3. 底部元素的position的值为absolute/fixed
    2. 父元素与子元素重叠
       1. 父元素加入(overflow: hidden);
       2. 父元素添加透明边框(border:1px solid transparent);
       3. 子元素变为行内盒子(display: inline-block);
       4. 子元素加入浮动属性或定位

## 垂直剧中

  已知高度和宽度的元素

  1. 设置父元素为相对定位，给子元素设置绝对定位，top: 0; right: 0; bottom: 0; left: 0; margin: auto;
  2. 设置父元素为相对定位，给子元素设置绝对定位，left: 50%; top: 50%; margin-left: --元素宽度的一半px; margin-top: --元素高度的一半px;

  未知高度和宽度的元素

  1. 设置父元素为相对定位，给子元素设置绝对定位，left: 50%; top: 50%; transform: translateX(-50%) translateY(-50%）
  2. 设置父元素为flex定位，justify-content: center; align-items: center;

  （relative 生成相对定位的元素，相对于其正常位置进行定位。）
  （absolute 生成绝对定位的元素，相对于 static 定位以外的第一个父元素进行定位。）

## flex弹性盒子布局

容器的属性

  主轴的方向
    flex-direction: row | row-reverse | column ｜ column-reverse
      row（默认值）：主轴为水平方向，起点在左端。
      row-reverse：主轴为水平方向，起点在右端。
      column：主轴为垂直方向，起点在上沿。
      column-reverse：主轴为垂直方向，起点在下沿。

  换行属性
    flex-wrap: nowrap | wrap | wrap-revers
      nowrap（默认）：不换行。
      wrap：换行，第一行在上方。
      wrap-reverse：换行，第一行在下方。

  简写：方向 + 换行
    `flex-flow：<flex-direction> || <flex-wrap>`

  主轴对齐方式
    justify-content: flex-start | flex-end | center | space-between | space-around;
      flex-start（默认值）：左对齐
      flex-end：右对齐
      center： 居中
      space-between：两端对齐，项目之间的间隔都相等。
      space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

  交叉轴对齐方式
    align-items: flex-start | flex-end | center | baseline | stretch;
      flex-start：交叉轴的起点对齐。
      flex-end：交叉轴的终点对齐。
      center：交叉轴的中点对齐。
      baseline: 项目的第一行文字的基线对齐。
      stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度
  
  多根轴线对齐方式
    align-content: flex-start | flex-end | center | space-between | space-around | stretch;
      flex-start：与交叉轴的起点对齐。
      flex-end：与交叉轴的终点对齐。
      center：与交叉轴的中点对齐。
      space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
      space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
      stretch（默认值）：轴线占满整个交叉轴。

  项目的属性
    排列顺序，数值越小，排列越靠前，默认为0。
      `order: <integer>`
    项目的放大比例,默认为0，即如果存在剩余空间，也不放大。
      `flex-grow: <number>`; /*default 0*/
    项目的缩小比例,默认为1，即如果空间不足，该项目将缩小
      `flex-shrink: <number>`; /*default 1*/
    项目占据的空间,默认值为auto，即项目的本来大小
      `flex-basis: <length> | auto`; /*default auto*/

## 重排（回流）

  定义：DOM中各个元素都有自己的盒子模型，需要浏览器根据样式进行计算，并根据计算结果将元素放到特定位置，这就是Reflow

  触发Reflow的条件

  1. 增、删、改、移DOM
  2. 修改CSS样式
  3. Resize窗口
  4. 页面滚动
  5. 修改网页的默认字体

## 重绘

   定义：当各种盒子的位置、大小以及其他属性改变时，浏览器需要把这些元素都按照各自的特性绘制一遍，这个过程称为Repaint

  触发Repaint的条件
  
  1. DOM改动
  2. CSS改动

  回流一定会触发重绘，而重绘不一定会回流

## 如何减少重绘、避免重排

  1. DOM层面：DocumentFragment本质上是一个占位符，真正插入页面的是它的所有子孙节点，所以，将需要变动的DOM节点先汇总到DocumentFragment，然后一次性插入，可以减少DOM操作的次数
  2. CSS层面：操作多个样式时，可以先汇总到一个类中，然后一次性修改

bfc（块级格式化上下文）
  是Web页面的可视CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域

  布局规则

  1. 内部的Box会在垂直方向，一个接一个地放置。
  2. Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠
  3. BFC的区域不会与float box重叠。
  4. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。
  5. 计算BFC的高度时，浮动元素也参与计算

## 哪些元素会生成BFC

  1. float不为none
  2. position不为static/relative
  3. display的值为inline-block、table-cell、table-caption
  4. overflow的值不为visible
  5. 根元素

## 分析比较 opacity: 0、visibility: hidden、display: none 优劣和适用场景

  display: none - 不占空间，不能点击，会引起回流，子元素不影响
  visibility: hidden - 占据空间，不能点击，引起重绘，子元素可设置 visible 进行显示
  opacity: 0 - 占据空间，可以点击，引起重绘，子元素不影响
