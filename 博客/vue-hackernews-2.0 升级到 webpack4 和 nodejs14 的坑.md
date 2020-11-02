## 概述

最近非常想做一个服务端渲染项目，那就打算从尤大的[vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0)开始入手呗。其实我之前试图改造过这个项目，但是因为当时很菜所以失败了。现在我觉得有能力改造好，那就开始呗。把心得记录下来，供以后开发时参考，相信对其他人也有用。

### mode

webpack4带来的第一个大变化就是增加了mode字段来区分生产环境还是开发环境，所以我们到```webpack.base.config.js```里面去增加mode字段：

```js
mode: isProd ? 'production' : 'development',
```

### package.json

老项目的依赖如下：

``` json
"dependencies": {
    "compression": "^1.7.1",
    "cross-env": "^5.1.1",
    "es6-promise": "^4.1.1",
    "express": "^4.16.2",
    "extract-text-webpack-plugin": "^3.0.2",
    "firebase": "4.6.2",
    "lru-cache": "^4.1.1",
    "route-cache": "0.4.3",
    "serve-favicon": "^2.4.5",
    "vue": "^2.5.22",
    "vue-router": "^3.0.1",
    "vue-server-renderer": "^2.5.22",
    "vuex": "^3.0.1",
    "vuex-router-sync": "^5.0.0"
},
"devDependencies": {
    "autoprefixer": "^7.1.6",
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-plugin-syntax-dynamic-import": "^6.18.0",
    "babel-preset-env": "^1.6.1",
    "chokidar": "^1.7.0",
    "css-loader": "^0.28.7",
    "file-loader": "^1.1.5",
    "friendly-errors-webpack-plugin": "^1.6.1",
    "extract-text-webpack-plugin": "^3.0.2",
    "rimraf": "^2.6.2",
    "stylus": "^0.54.5",
    "stylus-loader": "^3.0.1",
    "sw-precache-webpack-plugin": "^0.11.4",
    "url-loader": "^0.6.2",
    "vue-loader": "^15.3.0",
    "vue-template-compiler": "^2.5.22",
    "webpack": "^3.8.1",
    "webpack-dev-middleware": "^1.12.0",
    "webpack-hot-middleware": "^2.20.0",
    "webpack-merge": "^4.2.1",
    "webpack-node-externals": "^1.7.2"
}
```

我们删除一下插件，用更新的插件来代替他：

```js
// 删除
"babel-core": "^6.26.0",
"extract-text-webpack-plugin": "^3.0.2",

// 添加
"@babel/cli": "^7.11.6",
"@babel/core": "^7.11.6",
"@babel/preset-env": "^7.11.5",
"mini-css-extract-plugin": "^1.0.0",

// webpack4 需要安装 webpack-cli
"webpack-cli": "^3.3.12",
```

然后重新安装所有其它依赖，让它们达到最新版本。

### firebase

尤大的这个项目是使用 firebase 来获取 hackernews 的数据的，我本来想用 axios 取代的，但是我去看了一下[hackernews 的 api](https://github.com/HackerNews/API)之后发现，尤大的 firebase 的写法只需要少量代码就能够获取全部数据，如果使用 axios 的话，就需要先去请求一个总接口获取文章 id，然后再用这些 id 去一篇篇获取每篇文章的数据，非常麻烦，所以这里就没有升级到 axios。

另外，我去看了一下最新版本的 firebase 的 api，发现使用方法和尤大那个版本的使用方法是一样的，所以这里只需要升级 firebase 到最新版本即可。

### lru-cache

lru-cache 升级到最新版本之后需要使用 new 语法：

```
// 旧的语法
api.cachedItems = LRU({
    max: 1000,
    maxAge: 1000 * 60 * 15 // 15 min cache
})

// 新的语法
api.cachedItems = new LRU({
    max: 1000,
    maxAge: 1000 * 60 * 15 // 15 min cache
})
```

### babel

老项目使用的 babel-core 现在已经不建议使用了，用```@babel/preset-env```替代，然后在 babel-loader 的 presets 里面加入```@babel/preset-env```：

```js
{
    test: /\.js$/,
    loader: 'babel-loader',
    exclude: /node_modules/,
    options: {
        presets: ['@babel/preset-env']
    }
}
```

### css

webpack4 摈弃了 extract-text-webpack-plugin 而使用新的 mini-css-extract-plugin 插件，我们注释掉上一个插件的代码，然后使用新的 mini-css-extract-plugin 插件。

但是这个插件有一个问题，就是它在 extract css 文件的时候会使用 document 方法，从而导致 ssr 的服务端打包的时候会报```document is not defined```的错误，具体可以看[这里](https://github.com/webpack-contrib/mini-css-extract-plugin/issues/90)

解决方案推荐使用 null-loader，原理是在服务端打包的时候，不处理 css，这就要求客户端和服务端使用不同的配置，代码如下：

```js
// webpack.client.config.js
module: {
    rules: [
      {
        test: /\.styl(us)?$/,
        use: [
          isProd ? {
            loader: MiniCssExtractPlugin.loader,
            options: {
              esModule: false,
            },
          } : 'vue-style-loader',
          {
            loader: 'css-loader',
            options: {
              esModule: false,
            }
          },
          'stylus-loader'
        ],
      },
    ],
}
plugins: [
    new MiniCssExtractPlugin({
      filename: 'common.[chunkhash].css'
    }),
]

// webpack.server.config.js
module: {
    rules: [
      {
        test: /\.styl(us)?$/,
        use: 'null-loader'
      }
    ]
},
```

由于此项目是使用 stylus 的，所以如果使用 scss 等的话可以类似处理。

需要注意的是，**由于很多 loader 的升级，他们都默认开启了 esmodule 功能，我们需要关闭它（因为vue-style-loader还没有加入这个功能）。**

### CommonsChunkPlugin

老项目是使用 commonchunksplugin 抽取公共模块和css模块的，代码如下：

```js
// extract vendor chunks for better caching
new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks: function (module) {
    // a module is extracted into the vendor chunk if...
    return (
        // it's inside node_modules
        /node_modules/.test(module.context) &&
        // and not a CSS file (due to extract-text-webpack-plugin limitation)
        !/\.css$/.test(module.request)
    )
    }
}),
// extract webpack runtime & manifest to avoid vendor chunk hash changing
// on every build.
new webpack.optimize.CommonsChunkPlugin({
    name: 'manifest'
}),
```

而 webpack4 则直接内置了 optimization 字段来实现这个功能，所以我们注释掉上面的代码，然后加入如下代码：

```js
// webpack.client.js
optimization: {
    splitChunks: {
        chunks: 'all',
        cacheGroups: {
        styles: {
            name: 'styles',
            test: /css$/,
            enforce: true,
        },
        vendor: {
            name: 'vendor',
            test: /[\/]node_modules[\/]/,
            enforce: true,
        },
        },
    },
    runtimeChunk: {
        name: 'manifest',
    },
},

```

### externals

老项目是使用 externals 字段在服务端打包的时候防止 css 没有被引入，但是这里我们已经在服务端不打包 css 了，所以这个配置可以删去。但是这里我们加上如下配置，来防止引入 node_modules 里面的包，从而提高打包效率：

```js
// webpack.server.config.js
externals: Object.keys(require('../package.json').dependencies),
```

### service-worker

老项目是使用[sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin)来安装 service-worker 的，现在这个包已经被[workbox-webpack-plugin](https://www.npmjs.com/package/workbox-webpack-plugin)代替了，相关文档在[这里](https://developers.google.com/web/tools/workbox/modules/workbox-webpack-plugin)

我们首先卸载 sw-precache-webpack-plugin 然后安装 workbox-webpack-plugin：

```
// 卸载
"sw-precache-webpack-plugin": "^0.11.4",

// 安装
"workbox-webpack-plugin": "^5.1.4"
```

然后我们在```webpack.client.config.js```里面删掉 SWPrecachePlugin 的代码，加入 WorkboxPlugin 的相关配置：

```js
// webpack.client.config.js
const WorkboxPlugin = require('workbox-webpack-plugin');

if (process.env.NODE_ENV === 'production') {
  config.plugins.push(
    new WorkboxPlugin.GenerateSW({
      cacheId: 'vue-hn',
      swDest: 'service-worker.js',
      clientsClaim: true,
      skipWaiting: true,
      dontCacheBustURLsMatching: /./,
      exclude: [/\.map$/, /\.json$/],
      runtimeCaching: [
        {
          urlPattern: '/',
          handler: 'NetworkFirst'
        },
        {
          urlPattern: /\/(top|new|show|ask|jobs)/,
          handler: 'NetworkFirst'
        },
        {
          urlPattern: '/item/:id',
          handler: 'NetworkFirst'
        },
        {
          urlPattern: '/user/:id',
          handler: 'NetworkFirst'
        }
      ]
    })
  )
}
```

由于 service-worker 支持在 localhost 进行调试，所以我们更改一下老项目引入 service-worker 的代码：

```js
// entry-client.js
if ('serviceWorker' in navigator && process.env.NODE_ENV === 'production') {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js').then(registration => {
      console.log('SW registered: ', registration);
    }).catch(registrationError => {
      console.log('SW registration failed: ', registrationError);
    });
  });
}
```

虽然到这里就已经替换完成了，但是在实际打包之后，WorkboxPlugin 会生成一个带有hash的```workbox.js```文件，当我们更改 WorkboxPlugin 相关配置之后再打包，这个 hash 值会变化。这里面有一个问题就是，这个文件没有被 express 暴露出来，所以请求不到这个文件（express 只暴露了没带 hash 的 service-worker.js文件）。

解决方法有2种，第一种是直接改```entry-client.js```里面的```service-worker```路径，在前面加上 dist。这样```service-worker.js```在请求 workbox.js 的时候回自己带上 dist 前缀。代码如下：

```js
// entry-client.js
if ('serviceWorker' in navigator && process.env.NODE_ENV === 'production') {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/dist/service-worker.js').then(registration => {
      console.log('SW registered: ', registration);
    }).catch(registrationError => {
      console.log('SW registration failed: ', registrationError);
    });
  });
}
```

另一种解决方法是在 express 里面动态引入带有hash的```workbox.js```文件。由于这个文件带有 hash 值，所以它不能被 express.static 直接引入，app.use 也不支持正则引入，所以我们需要获取 dist 文件夹下面的所有文件名，找出带有 workbox.js 的文件名，然后利用这个文件名通过 express.static 暴露出来，代码如下：

```js
// workbox dir
const distFiles = fs.readdirSync('./dist')
const workboxDir = distFiles.filter(name => /workbox-.*\.js$/.test(name))

if (workboxDir.length > 0) {
  app.use(`/${workboxDir[0]}`, serve(`./dist/${workboxDir[0]}`))
}
```

### 完工

到这里就全部升级完毕了，详细代码可以参考[我的vue-hackernews-2.0项目](https://github.com/sishenhei7/vue-hackernews-2.0-webpack4)

我学到了什么：

1. 了解了一下 webpack 在 ssr 的各种配置
2. 尝试了一些 node api
3. 解决 bug 的能力，升级 api 的时候有各种bug，去 issue、stackoverflow、源码和其它类似项目中找解决方案。
4. 为以后编写[cheer-fun](https://github.com/cheer-fun/pixivic-pc)的项目做准备

