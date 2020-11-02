## 概述

最近做项目用flex重构了一下网页中的布局，顺便学习了一下flex弹性布局，感觉超级强大，有一些心得，记录下来供以后开发时参考，相信对其他人也有用。

参考资料：

[Solved by Flexbox](https://magic-akari.github.io/solved-by-flexbox/)

[Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

### flex基础

flex基础语法可以参考上面阮一峰的flex布局教程。简要如下：

```
display: flex;
justify-content: space-between; //子元素横向排列方式
align-items:center; //子元素纵向排列方式
```

注意：**父元素声明为flex之后，子元素不需要声明为flex。**

### 强大的flex: 1

在布局的时候，我们经常会遇到需要让子元素的宽度随着父元素的宽度改变的情况，即子元素需要自己撑满父元素。比如粘性页脚，让高度未知的页脚粘在高度未知的父元素的底部。这个时候只需要加上下面的css即可：

```
// 父元素声明为flex，排列方式为上下排列
.Site {
  display: flex;
  min-height: 100vh;
  flex-direction: column;
}

// 要自己撑满父元素的子元素加上这个class（不是页脚哦~）
.Site-content {
  flex: 1;
}
```

注意：**利用flex:1和父div包裹可以实现各种强大的布局。如果不行的话，就给它包一层flexbox的父元素轻松解决啦**~

### 深入flex: 1

flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。

该属性有两个快捷值：auto (1 1 auto) 和 none (0 0 auto)。

```
flex-grow属性定义了项目的 放大比例，默认为0，就是不放大。

flex-shrink属性则定义了项目的缩小比例，默认为1，就是如果空间不足的话，项目将缩小。

flex-basis定义了项目的本来大小，基本相当于width或者height。
```

注意：这里有一个坑，就是低版本浏览器在解析flex和width属性的时候会发生冲突，表现出来就像是flex-wrap不生效的样子，当初解决这个问题花了我3.5个小时。**所以一般对于flexbox不直接写width: 50%，而是用flex: 0 0 50%来代替；如果width是具体的值width: 200px，则用flex: 0 0 200px代替**。
