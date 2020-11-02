## 概述

这几天研究了一下在vue中使用svg sprite，有些心得，记录下来，供以后开发时参考，相信对其它人也有用。

### 在vue中导入svg

在vue中导入svg的方法有很多种，比如在img标签的src属性中导入，但是这样就不能使用class改变svg的颜色。所以一般利用svg的use标签使用内联svg的方法导入。例如下面：

```
<svg>
  <use xlink:href="@/assets/sprite.svg#notification"/>
</svg>
```

使用这种方法需要注意一点，就是如果你想把路径写成变量的形式，下面的写法是行不通的：

```
<svg>
  <use :xlink:href="'@/assets/sprite.svg#notification'"/>
</svg>
```

因为vue没有解析里面的字符串，所以不能生成所需要的路径，而且，即使能够解析里面的字符串，也由于没有加上hash值导致解析不了。

解决方法是使用require生成相对路径。示例如下：

```
<svg>
  <use :xlink:href="`${require('@/assets/sprite.svg')}#notification`"/>
</svg>
```

需要注意的是，使用require的下面写法是不行的。因为路径中的#解析不了。

```
<svg>
  <use :xlink:href="require('@/assets/sprite.svg#notification')"/>
</svg>
```

### 使用svg雪碧图

如果svg图标有很多的话，会发出很多http请求，资源消耗量很大。这个时候最好把svg做成雪碧图。一般情况下，我们手动把要做成雪碧图的svg文件编上id，全部放到一个svg文件比如sprite.svg文件里面去，然后用如下方式引用就可以了(其中notification是引用的svg的id)：

```
<svg>
  <use xlink:href="@/assets/sprite.svg#notification"/>
</svg>
```

上面的方法看起来很完美，但是有一个很严重的缺点，它的svg雪碧图是存在内存里面的，所以在切换路由的时候，svg雪碧图就没有了，需要重新下载这个svg雪碧图。很浪费资源啊。

### 改进方案

参考[svg-sprite-loader实现加载svg自定义组件](https://www.cnblogs.com/frost-yen/p/9608711.html)和[VUE-cli3使用 svg-sprite-loader](https://it.520mwx.com/view/31955)可以看到，可以利用svg-sprite-loader来做svg雪碧图。但是都需要修改webpack的loader配置。

### webpack-chain

vue-cli内部是利用[webpack-chain插件](https://github.com/neutrinojs/webpack-chain)修改webpack配置的，这是[源码](https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-service/lib/config/base.js)。外部也只能在vue.config.js里面利用webpack-chain来修改webpack配置。

具体的使用方法可以参考：[webpack-chain文档](https://github.com/neutrinojs/webpack-chain)和[webpack-配置-module](https://www.webpackjs.com/configuration/module/)

但是这里有一个坑，就是怎么按条件修改loader配置，比如在某个文件夹使用一种loader，在其它文件夹使用其它loader。看了半天文档，我最后发现，可以用oneOf来实现，其中oneOf(name)的name是自定义的，随便写语义化的名称就行。webpack-chain里面的oneOf和webpack配置里面的[oneOf](https://www.webpackjs.com/configuration/module/#rule-oneof)是对应的。实例如下：

```
module.exports = {
  chainWebpack: config => {
    const resolve = dir => path.join(__dirname, dir);
    const svgRule = config.module.rule('svg');
    svgRule.uses.clear();

    svgRule
      .test(/\.svg$/)
      .oneOf('normal')
      .exclude
        .add(resolve('src/assets/svg-sprite'))
        .end()
      .use('file-loader')
        .loader('file-loader')
        .end()
      .end()
      .oneOf('sprite')
      .include
        .add(resolve('src/assets/svg-sprite'))
        .end()
      .use('svg-sprite-loader')
        .loader('svg-sprite-loader')
        .options({
          symbolId: 'sprite-[name]'
        });
  }
};
```

还有一个坑就是end方法的使用，在适当的嵌套中需要加end方法返回上一层，否则后面的语句不会执行。

### 实现svg雪碧图

具体可以参考我写的[yi-svg-sprite插件](https://github.com/sishenhei7/yi-svg-sprite);

解释一下相关的原理和配置：

原理是利用svg-sprite-loader自动形成一个大的svg雪碧图内嵌到app的html里面，然后只需要在其它地方使用id引用里面的svg片段即可。

vue.config.js里面的操作是删除vue-cli里面对svg的loader处理，然后加上自己对svg的loader处理：在svg-sprite文件夹里面使用svg-sprite-loader，在其它文件夹里面使用file-loader(抄的vue-cli原配置);

main.js里面的操作是导入yi-svg-sprite库，并且把svg-sprite文件夹里面的svg文件组装成一个svg雪碧图，id是各自的文件名。

### 其它svg-sprite库

后来我才发现，已经有很多svg-sprite库了，下面对它们一一评价：

- [vue-svg-sprite](https://github.com/thierrymichel/vue-svg-sprite)：这个svg sprite库使用directive让人眼前一亮，但是它仍然有在切换路由的时候会重新加载svg雪碧图的缺点。
- [vue-cli-plugin-svg-sprite](https://github.com/swisnl/vue-cli-plugin-svg-sprite): 这个svg sprite库看起来很完美，也是包装的svg-sprite-loader，但是配置项太多，担心出现其它问题，而且好像没有维护了。
