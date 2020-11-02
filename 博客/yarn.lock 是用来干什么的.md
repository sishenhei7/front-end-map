## 概述

今天本地运行尤大的[vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0)，使用 yarn 命令安装，报错提示 node 版本必须大于7小于9，如下所示：

```
error upath@1.0.4: The engine "node" is incompatible with this module. Expected version ">=4 <=9". error Found incompatible module
```

然后我到 package.json 文件里面去看，明明只说了 node 要大于7，没有说要小于9啊。于是我去找 issue，发现竟[有人能够解决这个问题](https://github.com/vuejs/vue-hackernews-2.0/issues/329)。看了下他的解决方法，原来是 yarn.lock 的原因。

于是我查了下 yarn.lock 的资料，记录下来，供以后开发时参考，相信对其他人也有用。

### yarn.lock

官方对 yarn.lock 文件的说明如下：

```
为了跨机器安装得到一致的结果，Yarn 需要比你配置在 package.json 中的依赖列表更多的信息。 Yarn 需要准确存储每个安装的依赖是哪个版本。
为了做到这样，Yarn 使用一个你项目根目录里的 yarn.lock 文件。这可以媲美其他像 Bundler 或 Cargo 这样的包管理器的 lockfiles。它类似于 npm 的 npm-shrinkwrap.json，然而他并不是有损的并且它能创建可重现的结果。
```

需要注意的是：所有 yarn.lock 文件应该被提交到版本控制系统。

### npm-shrinkwrap.json

上面说到了 npm-shrinkwrap.json，那 npm-shrinkwrap.json 又是什么呢？

通过查资料，简单来说，它是由命令 npm shrinkwrap 生成的，它的作用和 package-lock.json 的作用是一样的，都是用来锁定版本用的。现在有了 package-lock.json 之后就不怎么用 npm-shrinkwrap.json 了。

### 解决问题

那么一开始我遇到的问题是什么原因呢？怎么解决呢？

再次摘抄报错提示如下：

```
error upath@1.0.4: The engine "node" is incompatible with this module. Expected version ">=4 <=9". error Found incompatible module
```

那就很简单了，原因是 upath@1.0.4 这个库只支持 >=4 <=9 的 node。那这个库 upath@1.0.4 锁定版本在 1.0.4 了，猜测是因为 yarn.lock 里面锁定了，所以去里面找，果然找到了。

所以解决方法有3个：

1. 修改 yarn.lock 文件，把 upath 的版本改成最新版本，再 yarn 一遍。
2. 删掉 yarn.lock 文件里面关于 upath@1.0.4 的信息，再 yarn 一遍。
3. 直接删掉 yarn.lock 文件，再 yarn 一遍。
