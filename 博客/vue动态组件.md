## 概述

最近用nuxt.js搞服务端渲染，研究了一下vue的动态组件，记录下来供以后开发时参考，相信对其他人也有用。

参考资料：

[vue.js 动态组件 & 异步组件](https://cn.vuejs.org/v2/guide/components-dynamic-async.html)

[vuejs 内置组件 component](https://juejin.im/entry/593e1087da2f6000672a20a4)

[Code Splitting](https://webpack.js.org/guides/code-splitting/)

### 原理

动态组件的原理是利用 webpack 的 code spliting 将动态加载的组件分块打包，然后当代码执行到这里的时候用ajax请求对应的地址。

### 方法

有如下三种方法实现动态组件：

方法一，利用resolve直接在单文件组件的component里面引入即可。

``` JavaScript
components: {
  PaymentBoard,
  PaymentForm: resolve => require(['@/components/payment/PaymentForm'], resolve),
},
```

方法二是利用import引入，这种方式其实就是方法一的语法糖

``` JavaScript
components: {
  PaymentBoard,
  PaymentForm: () => import('@/components/payment/PaymentForm'),
},
```

方法三是使用vue内置的component组件，在mounted阶段动态引入：

```
<!-- template -->
<component
  :is="componentName"
  :selectedCatsSum="selectedCatsSum"
  :selectedNegotiatedSecondCatsSum="selectedNegotiatedSecondCatsSum"
  :selectedNegotiatedThirdCatsSum="selectedNegotiatedThirdCatsSum"
  :selectedCatsDiscount="selectedCatsDiscount"
  :selectedCatsPrice="selectedCatsPrice"
  :selectedCats="selectedCats"
/>

// script
mounted() {
  this.componentName = () => import('@/components/payment/PaymentForm');
  const selectedCats = this.sessionStorageGet('selectedCats');
  if (selectedCats) this.selectedCats = selectedCats;
  this.init();
},
```

### 工厂函数形式的动态组件

还可以使用如下工厂函数形式的动态组件:

```js
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
});

components: {
  AsyncComponent,
},
```