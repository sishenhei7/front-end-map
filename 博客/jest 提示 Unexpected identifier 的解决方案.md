## 概述

今天在玩 jest 的时候，发现用 import 就会报 **Unexpected identifier** 的错误。查了很久的资料，最后终于解决了。

参考资料：[Jest tests can't process import statement](https://github.com/vuejs/vue-cli/issues/1584)

### 解决方案

1.首先需要安装下面2个库：

```
"babel-jest": "^23.6.0",
"babel-core": "7.0.0-bridge.0",
```

注意**版本号**一定要和上面的一样，如果版本太高，那就卸载然后重新安装。

2.删掉 node_modules 文件夹，然后安装 yarn ，用 yarn 来安装依赖。

经过试验，第二步**不可缺少**，只要用 npm 安装都不能解决问题。而且我看见 vue-cli 在安装的时候也会自动使用 yarn，虽然安装完成之后还是可以使用npm。

### 学到了什么

以后碰到这类关于 npm 包的问题，可以用 yarn 安装试试，反正安装之后仍然可以用 npm。


