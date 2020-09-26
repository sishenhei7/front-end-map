## CSS篇

### 字体

```
注意：手机上最小字体是8px，PC上最小字体是12px。不要使用奇数单位来定义字体大小，否则在低端设备上可能会产生模糊。
```

### 移动端 1px

```
产生原因：对于 1dpr 的屏幕，1px 等于 1 物理像素；而对于 2 dpr 的屏幕，1px 等于 2 物理像素。而人肉眼看到的是物理像素，所以在 dpr 高的移动端上面会看到线变宽了。

解决方案：一般根据伪元素 + 根据媒体查询进行缩放来实现。如果只兼容高版本 ios 的话，可以直接使用小数形式的 px 值；另外还可以使用 SVG + background-url 来实现。
```

### em、rem 和 vw

em:

```
定义：作为 font-size 单位时，代表父元素字体大小；作为其它单位时，代表自身字体大小；如果自身没有设置字体大小，则会查找父级，知道查找到浏览器设置的字体大小。

优点：解决了等比缩放下字体不能很好展示的问题

缺点：改变了父元素的字体大小之后，子元素可能会全部变化
```

rem:

```
定义：应用于非根元素时，相对于根元素字体大小；运用于根元素时，等比缩放

优点：解决了移动端的屏幕宽度兼容，本质上是对于不同屏幕宽度进行缩放

缺点：对于 PC 端兼容性不是很好；需要借助 js 动态修改根元素的字体大小；会丢失小数部分

参考：github上面的hotcss库——移动端布局解决方案
```

vw：

```
定义：1vw 就是屏幕宽度的 1%（这个单位也叫做 viewport 单位）

使用：postcss-px-to-viewport + postcss-loader

缺点：没有解决 1px 的显示问题；没有 rem 灵活
```

### 居中

行内元素：

```
// 父元素
text-align: center；

// 子元素
vertical-align: center;
```

固定宽高：

```
width: 200px;
height: 100px;
left: 50%;
top: 50%;
margin-left: -100px;
margin-top: -50px;
```

宽高不定：

```
// 使用 transform
transform: translate(-50%, -50%);

// 使用 table 布局
display: table-cell;
text-align: center;
vertical-align: middle;

// flex
display: flex;
justify-content: center;
align-items: center;

// grid
// 父元素
display: grid;
// 子元素
align-self: center;
justify-self: center;
```

### 布局

文档流：

```
定义：按照文档的顺序一个一个的显示，块元素独占一行，行内元素共享一行

脱离文档流会发生什么：会打乱元素的排序规则，而且不会撑开父元素了
```

表格布局：

```
display: table;
```

定位布局：

```
relative：没有脱离文档流，相对于自己在文档流中的位置
absolute: 脱离文档流，相对于设置了position为relative或者absolute或者fixed的父级元素
```

浮动布局：

```
定义：使用float属性，脱离文档流
应用：横向二列布局，横向三列布局
```

flex布局：

```
// 横向
justify-content: space-between;

// 纵向
align-items: center;

// 填充
flex: 1;
```

grid（网格）布局

```
教程：http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html
```

圣杯布局和双飞翼布局

```
本质上是通过浮动实现的，建议使用 flex 实现，更好
```

### BFC

```
定义：块级格式化上下文。文档流分为定位流、浮动流和普通流。BFC就是一个块级的普通流，它在里面有自己的一套渲染规则，决定了子元素如何布局，以及和其它元素之间的关系。
作用：清除内部浮动（display: block; clear: both;）；避免 margin 折叠；多列布局
```

### 命名规范

BEM 命名规范

```
定义：块(block) + 元素(element) + 修饰符(modifier)
优点：单一原则；模块化思想
```

OOCSS 命名规范（面向对象CSS）

```
理解：把不同元素里面通用的样式从模块、组件、对象中抽离出来，让它能在其它地方复用。
```

SMACSS 命名规范（结构化css）

```
把 css 分成了一下结构：
1.Base: reset.css
2.Layout: header、sidebar、i-header、i-sidebar等
3.Module：module、article、c-article等
4.Util：u-clearfix、u-ellipsis等
5.State：is-active、is-hidden等
6.Theme：button-large等
```

### Sass、Less、Stylus

```
特点：
1.选择符嵌套：反应层级和约束
2.变量/运算/函数：减少冗余代码
3.extend、mixin：代码片段重用，mixin更加强大
4.循环：适用于复杂有规律的样式
5.import：CSS 模块化
```

### 层叠上下文

```
定义：文档中的元素依据后来居上层叠在一起，浮动、z-index可以更改这种层叠顺序
```

### sticky

```
定义：sticky元素的相对偏移是相对于离他最近的具有滚动框的祖先元素，如果没有则是 viewport

注意：多个粘性元素共用一个粘性约束矩形的时候，滚动的时候会一个一个发生重叠
```

### line-height

```
理解：在inline-box模型中，每行文字都由一个line-box包裹，它的高度就是行高。它的高度决定了inline-box的高度。
```

### 盒模型

```
定义：盒模型是由内容、内边距、边框、外边距组成（盒子的宽度是指内容的宽度）
content-box：标准盒模型，盒子的宽度不包括边框和内边距（这个content就是内容）
border-box：怪异盒模型，盒子的宽度包含边框和内边距（这个border就是指包含border和padding）
```
