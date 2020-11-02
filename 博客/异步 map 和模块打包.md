## 概述

本文是我在查资料的时候**学到的一些东西**，记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[异步函数 - 提高 Promise 的易用性](https://developers.google.com/web/fundamentals/primers/async-functions)
[深入 CommonJs 与 ES6 Module](https://segmentfault.com/a/1190000017878394)
[聊聊 package.json 文件中的 module 字段](https://juejin.im/entry/5a99ed5c6fb9a028cd448d6a)

### 异步 map

我们知道，Array.map 能够遍历 Array 里面的每一个元素，然后组成一个新的数组，它的参数是一个函数。如果这个函数里面有异步方法会怎样呢？

``` js
const jsonPromises = urls.map(async url => {
  const response = await fetch(url);
  return response;
});
```

上面的例子中，在 Array.map 里面接了一个 async 函数，它不在乎我提供给它的是不是异步函数，只把它当作一个返回 Promise 的函数来看待。 也就是说：它不会等到第一个函数执行完毕就会调用第二个函数。

虽然上面的例子初看**很惊艳**，但是我**没有找到实际的应用场景**。。。

### 前端模块化

前端模块化发展这么久了，目前比较流行的模块化方案主要有下面三种：

1. umd 模块（通用模块）
2. CommonJs 模块（node模块）
3. es6 module（浏览器模块）

### umd 模块

使用 UMD 规范的 js 文件的**后缀通常写成 .umd.js**。

UMD 模块是以前比较流行的模块化方案，它的主要代码如下所示：

```js
(function (global, factory) {
  typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
  typeof define === 'function' && define.amd ? define(factory) :
  (global.libName = factory());
}(this, (function () { 'use strict';})));
```

这段代码判断运行环境，如果是 node 环境就使用 CommonJs 规范，如果不是则判断是否为 amd 环境，如果是则使用 AMD 规范，如果不是则导出为全局变量。所以 UMD 模块化是 **amd + commonjs + 全局变量** 这三种风格的结合。

因为 UMD 模块既能使用在浏览器环境，也能使用在 node 环境。所以现在需要在 html 中用 script 标签引入的包大多数都是用 UMD 模块。

### CommonJs

使用 CommonJs 规范的 js 文件的**后缀通常写成 .cjs.js**。

CommonJs 规范是 node 使用的模块体系，所有在 node 运行的包都是 CommonJs 规范。示例如下：

``` js
// foo.js
module.exports.foo = 2;

// index.js
const foo = require('./foo');
```

CommonJs 规范打包之后就变成类似下面这个样子：

```
(function (exports, require, module, __filename, __dirname) { ...
```

### es6 module

es6 module 规范又被称为 ESM 规范，使用这个规范的 js 文件的**后缀通常写成 .esm.js**。

CommonJs 规范是 我们开发的时候 使用的模块体系。示例如下：

```js
// foo.js
export default function () {}

// index.js
import foo from './foo';
```

### 静态规范与 tree shaking

我们经常听到别人说，CommonJs 模块规范是动态的，所以在任何时刻都能使用 require 引入包；而 ESM 模块规范是静态的，所以只能在最开始使用 import 引入包，而且会在编译的时候确定导入和导出。

下面解释下为什么 CommonJs 模块规范是**动态**的而 ESM 模块规范是**静态**的？

因为在 node 端，当我们 require 一个包的时候，我们只需要从磁盘里面加载这个包的内容即可，非常迅速；但是如果在浏览器端引入一个包的时候，需要从网络上发送 http 请求加载这个包，耗时非常久，会导致浏览器阻塞，所以不能当代码执行一半的时候引入包，而需要在顶层一次性把包全部引入。（而且是异步引入）

由于，ESM 模块需要在编译的时候引入包，这个缺点却带来一个意想不到的好处，就是 **tree shaking**。说白了，就是可以在编译的时候确定哪些代码被实际引入了，哪些没有，然后那些没有被实际引用的代码就不会被打包！！！这样可以极大的减少打包体积。

### pkg.module

但是我们在实际打包的时候，会忽略 node_modules 里面的内容，直接引入里面的包，而里面的包大多使用的是 UMD 规范，它在 node 端打包的时候是 CommonJs 规范，而 CommonJs 规范是动态的，编译的时候不知道哪些代码没有被使用，所以就**不能 tree shaking**。

这样打包 node_modules 里面的包的时候岂不是会打包很多无用的代码？？？

这个时候 pkg.module 就应运而生了，**pkg.module 指的是 package.json 文件中的 module 字段**。我们知道 package.json 文件中 main 字段指向的就是打包后的代码，这个代码一般是 UMD 规范。而 module 字段指向的是 ESM 规范编写的代码！！！这样我们在引入 node_modules 的时候，如果引用的是 module 字段指向的代码，我们就可以对这个包进行 tree shaking 了！！！
