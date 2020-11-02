## 概述

最近打算玩一下**service worker**，但是service worker只能在https下跑，所以查资料自己用纯express搭建了一个微服务器，把过程记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：[express官方文档](http://www.expressjs.com.cn/4x/api.html)

### http服务器

首先我们用express搭建一个**http服务器**，很简单，看看官方文档就可以搭建出来了。代码如下：

```
// server.js
const express = require('express');
const http = require('http');

const app = express();
const PORT = 7088; // 写个合理的值就好
const httpServer = http.createServer(app);

app.get('/', function (req, res) {
  res.send('hello world');
});

httpServer.listen(PORT, function () {
  console.log('HTTPS Server is running on: http://localhost:%s', PORT);
});
```

### 加入到项目中

我们的**理想状况**是，在项目目录下建立一个server文件夹，然后在server文件夹里面启动服务器，加载项目目录下的dist文件夹。

所以我们加入代码解析静态资源：

```
// server.js
const express = require('express');
const http = require('http');

const app = express();
const PORT = 7088; // 写个合理的值就好
const httpServer = http.createServer(app);

app.use('/', express.static('../dist'));

httpServer.listen(PORT, function () {
  console.log('HTTPS Server is running on: http://localhost:%s', PORT);
});
```

### 加入https

我们想把http变成https，首先我们要**生成本地证书**：

```
brew install mkcert
mkcert localhost 127.0.0.1 ::1
```

上面的代码意思是说，先安装mkcert，然后用mkcert给localhost，127.0.0.1和::1这三个域名生成证书。

然后我们可以在文件夹下面看到2个文件：

```
秘钥：example.com+3-key.pem
公钥：example.com+3.pem
```

我们在钥匙串里面把公钥添加信任。方法可参考：[在Vue里用Service Worker来搞个中间层（React同理）](https://www.colabug.com/3479278.html)

添加完之后我们把秘钥和公钥放在certificate文件夹，然后添加到credentials.js文件中，我们通过这个文件引入秘钥和公钥：

```
// credentials.js
const path = require('path');
const fs = require('fs');

// 引入秘钥
const privateKey = fs.readFileSync(path.resolve(__dirname, './certificate/example.com+3-key.pem'), 'utf8');
// 引入公钥
const certificate = fs.readFileSync(path.resolve(__dirname, './certificate/example.com+3.pem'), 'utf8');

module.exports = {
  key: privateKey,
  cert: certificate
};
```

最后我们把**http变成https**，并且引入秘钥和公钥：

```
// server.js
const express = require('express');
const https = require('https');
const credentials = require('./credentials');

const app = express();
const SSLPORT = 7081; // 写个合理的值就好
const httpsServer = https.createServer(credentials, app);

app.use('/', express.static('../dist'));

httpsServer.listen(SSLPORT, function () {
  console.log('HTTPS Server is running on: https://localhost:%s', SSLPORT);
});
```

### 设置api代理

在项目中，我们经常遇到跨域问题，在开发时我们是通过devServer的**proxyTable**解决的，而proxyTable在打包后是无效的。所以我们需要在服务器上面**代理api请求**。代码如下：

```
// proxy.js
const proxy = require('http-proxy-middleware');

const authApi = 'your-authApi-address';
const commonApi = 'your-commonApi-address';

module.exports = app => {
  app.use('/api/auth', proxy({
    target: authApi,
    changeOrigin: true,
    pathRewrite: {
      '/api/auth': '/auth'
    },
    secure: false,
  }));

  app.use('/api/common', proxy({
    target: commonApi,
    changeOrigin: true,
    pathRewrite: {
      '/api/common': '/api'
    },
    secure: false,
  }));
};
```

写法和devServer里面是一样的，因为devServer底层也是通过express实现的。

然后我们在server.js里面引入上面写的代理：

```
// server.js
const express = require('express');
const https = require('https');
const setProxy = require('./proxy');
const credentials = require('./credentials');

const app = express();
const SSLPORT = 7081; // 写个合理的值就好
const httpsServer = https.createServer(credentials, app);

app.use('/', express.static('../dist'));

setProxy(app);

httpsServer.listen(SSLPORT, function () {
  console.log('HTTPS Server is running on: https://localhost:%s', SSLPORT);
});
```

### 最后

最后我们把server.js，credentials.js和proxy.js放在一起就好了啦！

**用法：**只需要把整个文件夹放到项目目录，在里面运行下面的指令就好了：
```
yarn i
node server.js
```

详细代码可以参考[我的github](https://github.com/sishenhei7/httpsServer)