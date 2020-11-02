## 概述

今天打算快速使用vue-cli建立一个小应用用于测试，使用axios发送http请求，但是遇到了**跨域问题**，总结了一下，供以后开发时参考，相信对其他人也有用。

### vue-cli的跨域设置

在vue.config.js里面的devServer的**proxy**加入如下设置。

```
// vue.config.js
const tableauApi = 'https://tableau.proxy.web.yimian.com.cn/';

module.exports = {
  devServer: {
    proxy: {
      '/tableau': {
        target: tableauApi,
        changeOrigin: true,
        pathRewrite: {
          '^/tableau': ''
        },
      },
    },
  },
};
```

上面的设置表示，把```/tableau```开头的api代理到```https://tableau.proxy.web.yimian.com.cn/```，并且去掉/tableau。比如```/tableau/test1```就会被代理到```https://tableau.proxy.web.yimian.com.cn/test1```。

这里底层使用的是[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware#proxycontext-config)插件。

### 后续

1. 上面的设置中有devServer，表示只能在**开发环境中**使用代理，而打包之后就无效了。打包之后需要使用nginx进行反向代理才行。
2. 上传到静态服务器上面之后有一个路径问题，需要用**publicPath**给js, css等文件的路径添加前缀，我自己的设置如下：

```
// vue.config.js
module.exports = {
  publicPath: process.env.NODE_ENV === 'production' ?
    '/test/tableau/dist/':
    '/'
};
```

