## 概述

我们有很多打包工具，比如 webpack、rollup等等。但是如果我只想打包一个 js 文件呢？用他们会不会太重度了？其实完全没必要，只使用[babel](https://www.babeljs.cn/docs/usage)就可以打包了。

很多小型库都是这样打包的，比如：[file-loader](https://github.com/webpack-contrib/file-loader/blob/master/package.json)，[css-loader](https://github.com/webpack-contrib/css-loader/blob/master/package.json)

### 方法

先安装```@babel/core```和```@babel/cli```，然后直接在```package.json```的```scripts```里面添加下面内容就行了：

```
"build": "babel src -d dist --copy-files"
```

注意：这里的 --copy-files 表示对于那些 babel 处理不了的文件（比如md、png文件等），直接把它们复制到目标文件夹。

### 实例

比如我有一段代码:

``` js
// loader.js
import { getOptions } from 'loader-utils';

export default function loader(source) {
  const options = getOptions(this);

  source = source.replace(/\[name\]/g, options.name);

  return `export default ${ JSON.stringify(source) }`;
}

```

打包后就变成了：

``` js
"use strict";

Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = loader;

var _loaderUtils = require("loader-utils");

function loader(source) {
  const options = (0, _loaderUtils.getOptions)(this);
  source = source.replace(/\[name\]/g, options.name);
  return `export default ${JSON.stringify(source)}`;
}
```

很显然，ES Module 被打包成了 ejs，并且加了一个```__esModule: true```属性，和我们预想的一样。

### 保留 module

上面打包时的 babel 配置是：

``` js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: "current"
        }
      },
    ],
  ],
};
```

如果要保留 module，则配置如下：

``` js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: "current"
        },
        modules: false,
      },
    ],
  ],
};
```
