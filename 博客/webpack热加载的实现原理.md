## 概述

最近面试被问到了 webpack 热加载的实现原理，所以去研究了一下，记录下来供以后开发时参考，相信对其它人也有用。

### 热加载原理

这一部分我没有去看源码，只是看了别人的分析理清了一下思路，参考资料：


[Webpack HMR 原理解析](https://zhuanlan.zhihu.com/p/30669007)

[从零实现webpack热更新HMR](https://juejin.im/post/6844904020528594957)


主要流程如下：

1.首先 webpack-dev-server 会建立一个服务器，并且和浏览器建立 websocket 通信。

2.服务器监听文件变化，当文件变化的时候，会重新打包相应的 chunk，然后向浏览器发射 hash 和 ok 事件，通知浏览器对应的 chunkid 等信息。

3.浏览器监听 hash 和 ok 事件，再接受信息之后，通过 jsonp 向服务端请求对应的热更新代码。

4.最后浏览器把 jsonp 获得的代码注入到 html 的 head 里面去执行，从而实现了对应的模块替换。

### vue-loader 是怎么热更新的

[vue-loader](https://github.com/vuejs/vue-loader/blob/6a05115ddf3ea680ab2b00862b2891da2e98a41c/lib/codegen/hotReload.js)的源码如下：

``` js
const hotReloadAPIPath = JSON.stringify(require.resolve('vue-hot-reload-api'))

const genTemplateHotReloadCode = (id, request) => {
  return `
    module.hot.accept(${request}, function () {
      api.rerender('${id}', {
        render: render,
        staticRenderFns: staticRenderFns
      })
    })
  `.trim()
}

exports.genHotReloadCode = (id, functional, templateRequest) => {
  return `
/* hot reload */
if (module.hot) {
  var api = require(${hotReloadAPIPath})
  api.install(require('vue'))
  if (api.compatible) {
    module.hot.accept()
    if (!api.isRecorded('${id}')) {
      api.createRecord('${id}', component.options)
    } else {
      api.${functional ? 'rerender' : 'reload'}('${id}', component.options)
    }
    ${templateRequest ? genTemplateHotReloadCode(id, templateRequest) : ''}
  }
}
  `.trim()
}
```

可以看到，它其实在编译单文件组件的时候，会把热加载相关的代码**注入**到编译后的代码里面去，从而实现热更新的。

### vue-loader 热更新的状态怎么保留

通过[vue-loader说明](https://github.com/vuejs/vue-loader/blob/5a7d56d76ba81b74e901f7f294b031c861411861/docs/zh/guide/hot-reload.md)，我们可以看到对于状态的保留规则：

1.当编辑一个 template 的时候，组件会**重新渲染**。（rerender）

2.当编辑一个 script 的时候，组件会就地销毁并**重新创建**。（reload）

3.当编辑一个 style 的时候，通过**vue-style-loader**自行热重载，并不影响状态。

### vue-loader 源码里面的 api 是什么

里面的 api 其实是 [vue-hot-reload-api](https://github.com/vuejs/vue-hot-reload-api)，这个库其实就控制了组件在重新渲染（rerender）和重新创建（reload）的时候做的事。

### module.hot.accept

通过上面的源码可以看到，```module.hot.accept```其实并没有传递参数，这是什么用法呢？通过查阅[webpack官方文档](https://webpack.js.org/api/hot-module-replacement/#accept)，这个通常写在对应的模块里面，作用是**标明这个模块可以热加载**，在这个模块改变的时候会热加载这个模块。

所以，vue-loader 的热重载原理其实是：打包后的单文件组件在修改之后，会借用 devserver 的热重载功能更新单文件组件，并且通过监听```module.hot.data```的变化，使用```module.hot.accept```中的 api 对组件进行重新渲染或者重新创建。


