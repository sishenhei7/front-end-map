## 概述

今天调研了一下vue项目怎么向小程序迁移，有些心得，记录下来，供以后开发时参考，相信对其他人也有用。

基本上vue项目向小程序迁移不外乎2种方法，一种是用小程序的[web-view组件](https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html?search-key=userAgent)，另一种是用mpvue重新开发一个。第二种成本太高，所以我这里调研的基本上是第一种方法。

### 解决方案

对于一般项目来说，直接在小程序中给web-view组件的src填入vue项目的地址即可。但是web-view组件有如下限制：

1.web-view组件的src**不能动态变化**。这个限制基本可以无视，因为我们是单页面，不需要经常改变web-view的src。但是有些特殊情况需要考虑这个限制，比如我们使用setData方法通过判断url的参数来加载web-view的src是**行不通**的。

2.小程序的page中只能存在**一个**web-view。这个限制也可以无视，因为我们只需要一个web-view就行了。

3.web-view里面的项目，不能用window.open(xx, '_blank')，但是可以用window.open(xx, '_self')。

我们项目中有用到window.open(xx, '_blank')，所以我们需要判断是不是小程序环境，是的话就把window.open(xx, '_blank')改成window.open(xx, '_self')。判断方法有如下2种:

```
// 方法1(推荐)——要求：微信版本>=7.0.0
// 通过判断userAgent中包含miniProgram字样来判断小程序web-view环境。
export function isMiniProgram() {
  return !!navigator.userAgent.match(/miniProgram/i);
}

// 方法2——无要求
// 引入微信sdk，然后用微信sdk判断
npm install weixin-js-sdk --save

wx.miniProgram.getEnv(function(res) {
  console.log(res.miniprogram); //true
});
```

基本上按照上面的方法就可以使用web-view迁移vue项目到小程序中了。

### 其它

在调研过程中我还踩了一个其它的坑，这里也记录下来。

1.使用微信sdk的postMessage传的值必须是**字符串**，所以对于对象来说需要先用JSON.stringify处理一下。

```
wx.miniProgram.postMessage({ data: JSON.stringify(navigator.userAgent) });
```

2.web-view的bindmessage属性可以接受postMessage传递过来的值，但是它只会在**特定时机**（小程序后退、组件销毁、分享）才触发。所以不能期望，马上传完值，web-view就能马上接收到并做出响应。

3.官方文档对于[navigateTo](https://developers.weixin.qq.com/miniprogram/dev/api/wx.navigateTo.html?search-key=navigate&q=)的描述有误。传递的值并不在options.query里面，而是**就在options里面**。

```
wx.navigateTo({
  url: 'test?id=1'
})

// test.js
Page({
  onLoad(options) {
    console.log(options.id)
  }
})
```
