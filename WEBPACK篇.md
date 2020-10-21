## Webpack篇

### 基础

entry

```
定义：入口起点，指示 Webpack 应该使用哪个模块，来作为其内部构建依赖图的开始。单入口使用 string，多入口使用对象或者数组。
```

output

```
定义：告诉 webapck在哪里输出它所创建的bundles，以及如何命名这些文件，默认值为dist。

option:
path: 输出目录path的绝对路径
filename: 输出文件的文件名
publicPath: 项目中所有资源所指定的公共路径
library: webpack不仅可以打包应用程序，还可以用来打包 js library，所以这个字段就是你打包 library 时别人需要引入的模块名
libraryTarget: 在打包 js library 的时候，包使用的模块规范，比如 umd、cjs等等
```

loaders

```
定义：webpack 的转换器，负责把某种文件格式的内容转换成 webpack 可以打包的模块。
常用loaders: css-loader、less-loader、scss-loader、file-loader、url-loader、babel-loader、ts-loader、eslint-loader、thread-loader
```

plugins

```
定义：插件是比 loaders 更强的配置，它可以执行范围更广的任务，从打包优化和压缩，一直到重新定义环境中的变量。
HotModuleReplacementPlugin: 模块热更新插件
HtmlWebpackPlugin: 生成html
clean-webpack-plugin: 清理上一次项目生成的 bundle 文件
mini-css-extract-plugin: 抽离 css 提取为单独的文件
purgecss: 去除没有用过的css代码
optimize-css-assets-webpack-plugin: 使用压缩来减少打包后的体积
```

devtool

```
空： 不生成 source map
source-map: 生成独立的source map文件
hidden-source-map: 让浏览器不会自动加载 source map
inline-source-map: 把 source map 以 base64 格式内嵌到 javascript 中
cheap-source-map: 更快的生成 source map，里面没有列的信息
```

tree-shaking:

```
定义：消除项目中没有用到的代码
原理：依赖于es6模块的静态分析

// 开发环境开启：
mode: 'development',
optimization: {
    usedExports: true,
}

// 生产环境开启：
mode: 'production',
```

热更新

```
定义：允许项目在运行时更新修改的模块，而不需完全刷新

开启：在dev server里面使用hot: true开启，然后在项目里面通过判断module.hot来进行热更新

注意：各个框架的loader里面已经内置了热更新，不需要重复设置。比如 vue-loader 就内置了对 vue 的热更新
```

别名

```
定义: 通过resolve.alias配置
```

require 和 import

```
require: 是一个拷贝，可以放在任何地方，运行时加载
import: 是一个引用，需要放在最前面，编译时加载
```

如何解决循环引用：

```
commonJS: commonJS是在运行的时候加载模块的，每当进入一个模块，就在缓存里面准备一个空对象作为导出结果，模块加载完毕的时候会更新这个导出结果。所以如果中途再次加载这个模块时，会直接从缓存里面拿导出结果。

ES module: ES module是在编译的时候加载模块的，在进入一个模块的时候，会标记为 fetching，此时这个模块没有任何导出（可以认为导出 undefined）。所以如果中途再次加载这个模块时，拿到的结果其实是undefined。

那如何解决这种代码的不一致呢：

commonJS: 把exports放到最上面执行
ES module: 由于读取的值是动态的，所以我们可以把内容包在一个函数里面导出，在执行的时候准备好就可以了。
```

### 性能优化

打包分析：

```
webpack-bundle-analyzer：打包分析神器，底层使用 webpack 官方提供的 stats.json
speed-measure-webpack-plugin: 分析打包总耗时，帮助我们定位到可以优化 webpack 的配置
```

体积优化：

```
js压缩：使用mode: production会默认开启webpack4的压缩，其内部是通过terser-webpack-plugin这个插件实现的，它有parallel参数，可以多进程压缩。

css压缩：使用optimize-css-assets-webpack-plugin插件压缩css，其内部使用的压缩引擎是cssnano（首先使用mini-css-extract-plugin提取css）

去掉无用css：使用purgecss-webpack-plugin插件，需要配合mini-css-extract-plugin使用

图片压缩：手动通过tinypng来压缩图片；或者通过image-webpack-loader插件来完成，其内部基于imagemin这个node库来压缩图片的
```

代码分离：

```
入口起点：使用 entry 的 vendor 属性和 CommonsChunkPlugin 配置手动地分离代码
防止重复：使用 commonChunkPlugin 去重和分离 chunk
动态导入：通过import来实现代码的动态导入
treeshaking: 开启treeshaking
```

文件指纹：

```
策略：js文件使用 chunkhash，提取出来的 css 文件使用 contenthash
hash: 只要项目文件有修改，整个项目构建的hash值就会更改
chunkhash：不同的entry会生成不同的chunkhash
contenthash: 根据文件内容来定义 hash，文件内容不变，contenthash 不变
```

polyfills:

```
定义：把polyfills代码放到一个单独的文件import并且单独打包，然后在index.html里面通过判断进行按需加载。
```

利用多线程：

```
tread-loader: 使用 thread-loader 开启多线程
```

其它：

```
预编译模块：使用 DllPlugin 进行预编译
插件缓存：babel-loader插件使用缓存
sourcemap：使用合理的sourcemap
缩小构建目标：对一些loader使用exclude等属性
```

### webpack4

新特性：

```
1.环境支持，需要node的版本在8.9.4以上
2.零配置
3.删除了一些插件，内置了一些插件
4.生产环境下会自动使用一些功能，比如 treeshaking
5.开箱即用 WebAssembly
6.新的插件系统
```

### loader 和 plugin

1.loader 用于对模块的源代码进行转换，它可以使我在加载的时候预处理文件。plugin 则用于解决 loader 不能解决的所有事。

2.loader 的原理：loader其实就是一个函数，它接收一个 source 参数（就是 resource file），然后返回 1 个或 2 个参数，第一个参数是是字符串表示的 js 代码，第二个参数是 sourceMap。

3.plugin 的原理：plugin 其实就是一个有 apply 方法的类。在这个方法里面使用 tap 或者 tapAsync 进入 webpack 相应的事件钩子，然后使用 webpack 的内部数据做一些处理，在处理完之后调用 webpack 的 callback 方法即可。

### 前端工程

1.前端工程包括：webpack、git提交规范（husky、conventional-changelog）、代码规范（ts、eslint、prettier）、CI（github actions）、test（e2e test、unit test）、docs（vuepress）
