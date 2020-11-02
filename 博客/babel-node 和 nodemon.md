## 概述

今天我继续完善我做的用来 mock 前端数据的库：[ym-mock](https://github.com/sishenhei7/ym-mock)。

我想要实现 2 个需求：

1. 支持 es6，至少要能 import 吧。
2. 修改了代码之后能自动热更新，不能我修改了服务器代码要手动重启吧。

最后通过查阅资料，用 [babel-node](https://www.npmjs.com/package/@babel/node) 和 [nodemon](https://www.npmjs.com/package/nodemon) 实现了，我把方法记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[[译]使用Babel7+nodemon打造你的Node.js项目开发](https://juejin.im/post/5cbe734ff265da036b4a649c)

### babel-node

使用 babel-node 可以在 node 端**自行编译并运行 es6 甚至 es7**。安装方法如下：

```
npm i @babel/core @babel/cli @babel/preset-env @babel/node -D // 或者使用 yarn
```

注意：我这里是局部安装的，全局安装的方法请自行看官方文档。

然后我们需要在项目的根目录下面创建 .babelrc 文件：

```
// .babelrc
{
  "presets": ["@babel/preset-env"]
}
```

最后修改 package.json，使用 babel-node 启动服务器入口文件即可：

```
// 使用命令 npm run server 即可运行
"scripts": {
  "server": "babel-node server.js"
},
```

这里有 2 点需要说明一下：

1. 为什么要用 babel-node 而不用 @babel/register 或者 @babel/polyfill 库？因为后者**只能用于打包过程**。也就是说，需要先编译，然后才能运行。
2. babel-node 只是用于**非打包过程**的，如果需要打包的话（比如用于**生产环境**），则不建议使用 babel-node，因为 babel-node 的打包体积会非常大。

### nodemon

使用 nodemon 可以**监听文件修改，然后让服务器自行重启**。

首先我们安装 nodemon：

```
npm i nodemon -D // 或者使用 yarn
```

最后修改一下 package.json 的命令即可：

```
// 使用命令 npm run server 即可运行
"scripts": {
  "server": "nodemon --exec babel-node server.js"
},
```

说明一下为什么要加 **--exec 这个参数**：这个参数是让 nodemon 运行非 node 程序的，比如运行 py 文件```nodemon --exec "python -v" ./app.py```。在这里因为我们是用 nodemon 运行 babel-node，而不是 server.js，所以需要加 --exec 这个参数。
