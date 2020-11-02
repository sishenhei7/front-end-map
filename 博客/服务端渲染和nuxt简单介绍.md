## 概述

最近研究了一下服务端渲染，有一些心得，记录下来供以后开发时参考，相信对其他人也有用。

参考资料：

[Vue SSR指南](https://ssr.vuejs.org/zh/guide/#%E4%B8%8E%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%9B%86%E6%88%90)

[nuxt.js官网](https://zh.nuxtjs.org/guide)

### 服务端渲染介绍

服务端渲染简单来说，就是分别对项目用webpack打包2次，一次生成**vue-ssr-server-bundle.json**，一次生成**vue-ssr-client-manifest.json和其它静态文件**，最后用node搭一个服务器接收这2个json文件进行组装，并发送给用户。其中有以下几点需要注意：

1.工厂函数

我们需要对vue, vue-router, vuex用一个**工厂函数**包裹起来，进行延迟执行。原因是node server每次会接受很多http请求，而vue却只有一个示例，如果在打包前把实例先初始化的话，以后的每次请求就会发送同一个实例，造成交叉请求状态污染 (cross-request state pollution)。示例如下：

``` js
// app.js
const Vue = require('vue')

module.exports = function createApp (context) {
  return new Vue({
    data: {
      url: context.url
    },
    template: `<div>访问的 URL 是： {{ url }}</div>`
  })
}
```

2.服务端渲染的生命周期

在所有vue的生命周期钩子函数中，**beforeCreate 和 created** 会在服务端渲染的时候被调用，其它生命周期会在客户端执行。

所以我们在 beforeCreate 和 created 这2个生命周期内不能访问this，也不能访问window，更不能访问组件实例等。一般我们在 beforeCreate 和 created 中会做的事情有：发送http请求事先填充store，鉴权，发送http请求初始化组件data等。我们在客户端进行初始化的http请求都需要移动到 mounted 或者 beforeMount 生命周期中。

3.axios

我们发送http请求的库推荐使用axios。又因为axios不仅会用在客户端发送http请求，还会用在**服务端**发送http请求，所以在给axios设置拦截器的时候需要小心使用和window或者dom相关的方法。

4.HTML结构

因为vue的服务端渲染主要是由vue-server-renderer库完成的，它在解析html标签的时候会有[一些坑](https://ssr.vuejs.org/zh/guide/hydration.html#%E4%B8%80%E4%BA%9B%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E5%9D%91)，就是html结构需要**很严格**的书写，至少要做到以下2点：

1. 不要漏写结束标签。
2. p标签里面不能使用块级标签，如果非要使用的话，需要把p标签换成div标签。

5.缓存

我们一般给node server使用 micro-caching 缓存策略：让 node server 把动态内容储存1秒，也就是说无论这一秒有多少请求，node server 只会生成一次动态内容。这个是通过LRU库来实现的。另外还有组件级别的缓存，这里就不多说了。

### nuxt.js

1.yarn

虽然nuxt项目可以用npm运行，但是仍然推荐使用yarn来运行此项目，步骤如下：

先检查电脑中是否有homebrew:

``` bash
$ brew -v
```

如果有homebrew的话跳过此步，没有的话使用以下命令安装：

``` bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

最后使用homebrew安装yarn:

``` bash
$ brew install yarn
```

这里是npm和yarn的对照：

- ```npm install``` === ```yarn```
- ```npm run dev``` === ```yarn run dev```
- ```npm init``` === ```yarn init```
- ```npm install <package>``` === ```yarn add <package>```
- ```npm uninstall <package>``` === ```yarn remove <package>```
- ```npm update <package>``` === ```yarn upgrade <package>```

2.指令

``` bash
# 安装所有依赖包
$ yarn

# 安装某个依赖包
$ yarn add <package>

# 打开开发环境
$ yarn dev

# 正式环境
$ yarn build
$ yarn start

# 开发环境下重启服务(很重要)
$ 输入rs 再按回车键

# 自动修复eslint错误
$ yarn lint

# 生成可视化依赖图（相当于vue-cli3的vue inspect指令）
$ yarn analyze
```

3.在开发的时候需要注意如下几点：

1. eslint在一些方面比其它项目更加严格，按报错提示修改就可以了。（这些方面我没有改成和其它项目一样是因为我觉得这些习惯很好）
2. 静态资源放在static文件夹里面，svg雪碧图放在```assets/sprite/```文件夹里面。
3. 文件夹名字不能改。（nuxt.js本身要求）
4. 其它比如pages文件夹里面每个文件就是一个路由，可以看[nuxt.js官方文档](https://zh.nuxtjs.org/guide)。
