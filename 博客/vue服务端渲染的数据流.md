## 概述

面试中被问到了服务端渲染的数据流动，没答上来，复盘整理一下，供以后工作时参考，相信对其他人也有用。

[vue ssr 指南](https://ssr.vuejs.org/zh/guide/data.html#%E5%B8%A6%E6%9C%89%E9%80%BB%E8%BE%91%E9%85%8D%E7%BD%AE%E7%9A%84%E7%BB%84%E4%BB%B6-logic-collocation-with-components)

### 数据流动

主要分以下几种情况：

1.初次打开页面时，会在服务端获取 matched 路由，然后执行他们的 asyncData 方法，这个方法会把获取的数据存放到 vuex 的 store 里面去，在全部获取完成之后把数据附加到渲染上下文中，状态将自动序列化为`window.__INITIAL_STATE__`，并注入 HTML，在客户端挂在应用之前，store就获取到了状态：

```
// entry-server.js
import { createApp } from './app'

export default context => {
  return new Promise((resolve, reject) => {
    const { app, router, store } = createApp()

    router.push(context.url)

    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents()
      if (!matchedComponents.length) {
        return reject({ code: 404 })
      }

      // 对所有匹配的路由组件调用 `asyncData()`
      Promise.all(matchedComponents.map(Component => {
        if (Component.asyncData) {
          return Component.asyncData({
            store,
            route: router.currentRoute
          })
        }
      })).then(() => {
        // 在所有预取钩子(preFetch hook) resolve 后，
        // 我们的 store 现在已经填充入渲染应用程序所需的状态。
        // 当我们将状态附加到上下文，
        // 并且 `template` 选项用于 renderer 时，
        // 状态将自动序列化为 `window.__INITIAL_STATE__`，并注入 HTML。
        context.state = store.state

        resolve(app)
      }).catch(reject)
    }, reject)
  })
}
```

2.当在浏览器端改变当前路由的 params 的时候，直接请求本路由下面所有的 asyncData 方法，并更新 store。

```
// entry-client.js
// a global mixin that calls `asyncData` when a route component's params change
Vue.mixin({
  beforeRouteUpdate (to, from, next) {
    const { asyncData } = this.$options
    if (asyncData) {
      asyncData({
        store: this.$store,
        route: to
      }).then(next).catch(next)
    } else {
      next()
    }
  }
})
```

**注意**：beforeRouteUpdate这个钩子在当前路由改变，当时该组件被复用时调用。（初次加载的时候不会执行这个钩子，因为组件没有被复用）

3.挡在浏览器端从一个路由跳到另一个路由时，通过比较前后匹配到的路由，对于不同的路由，请求它们下面的 asyncData 方法，并更新 store。

```
// entry-client.js
router.beforeResolve((to, from, next) => {
    const matched = router.getMatchedComponents(to)
    const prevMatched = router.getMatchedComponents(from)
    let diffed = false
    const activated = matched.filter((c, i) => {
      return diffed || (diffed = (prevMatched[i] !== c))
    })
    const asyncDataHooks = activated.map(c => c.asyncData).filter(_ => _)
    if (!asyncDataHooks.length) {
      return next()
    }

    bar.start()
    Promise.all(asyncDataHooks.map(hook => hook({ store, route: to })))
      .then(() => {
        bar.finish()
        next()
      })
      .catch(next)
})
```

**注意**：beforeResolve这个钩子是全局解析守卫，会在路由变化时调用，并且在所有组件内守卫和异步路由组件被解析之后才调用。（初次加载的时候也不会加载这个钩子，因为这个钩子是在router.onReady之后才加入的）

### asyncData 和 beforeMount

在客户端有 2 种方法来获取数据，它们都能够随意使用，但是用户体验不同：

1.在 asyncData 里面获取，就像上面所介绍的，它会使 app 在拿到数据后才跳转路由。所以在拿到数据的过程中会开启 progressbar 来告诉用户正在发请求。

2.在 beforeMount 里面获取，它会先跳转路由，然后再获取数据。所以在跳转路由之后，需要在展示的位置放一个 spinner 来告诉用户正在加载数据。
