## 概述

最近调研了一下 tableau extensions 的实现，有一些心得，记录下来，供以后开发时参考，相信对其他人也有用。

总的来说，写 tableau extensions 还是**挺简单的**，只是要熟悉一点 tableau 的**业务逻辑**。这是我仿照 Image Viewer 插件写的 [轮播图插件](https://github.com/sishenhei7/my_tableau_extension)

参考资料：

[extensions-api](https://github.com/tableau/extensions-api)
[extensions-api 文档](https://tableau.github.io/extensions-api/docs/trex_getstarted.html)

### 原理

tableau extensions 的原理其实就是在 tableau 里面打开一个内嵌网页而已，而在这个网页里面可以接入 tableau 的 sdk 来和 tableau dashboard 进行数据交互。

需要注意的是：

1. 内嵌网页的内核是 **chrome**。
2. tableau extensions 只能在 **tableau 的仪表盘**里面运行。
3. 由于插件其实是一个**内嵌网页**，所以在插件里面你可以使用任何库，包括 jquery，vue，react 等等。

### 可以做哪些事

由于 tableau extensions 可以使用 sdk 和 tableau 进行数据的双向流动，所以它非常灵活，可以做这些事情：

1. 能够**获取** tableau dashboard 的数据
2. 能够**操作** tableau dashboard 的数据
3. 能够监听关于数据的操作事件，然后做出一些响应。（所以能够和 tableau dashboard 的其它模块的一些操作进行**联动**）
4. 能够打开 tableau dashboard 的**配置框**，让用户给自己手动进行一些配置。

### 基本结构

插件的基本结构如下(以我开发的[轮播图插件](https://github.com/sishenhei7/my_tableau_extension)为例)：

```
- ImageSwiper.trex
- imageswiper.css
- imageswiper.html
- imageswiper.js
- imageswiperDialog.html
- imageswiperDialog.js
```

插件的加载过程如下：

1. 安装插件: 引入配置文件 ImageSwiper.trex
2. 在 ImageSwiper.trex 文件中有 imageswiper.html 的加载地址，tableau 会利用这个地址加载 imageswiper.html
3. imageswiper.html 会加载 imageswiper.css 和 imageswiper.js
4. 如果有配置 tableau 的配置框的话，在 imageswiper.js 里面会引入 imageswiperDialog.html 的地址，然后 tableau 根据这个地址加载配置框 imageswiperDialog.html
5. imageswiperDialog.html 会加载 imageswiperDialog.js

所以要**查看其它 tableau extensions 的源码是很简单的**，只需要在 trex 配置文件里面查看插件 html 的地址，然后一步步去看源码就可以了。

### 调试

我当时是在 tableau desktop 上面写插件的，这样就看不到插件打印出来的内容，所以当时调试得非常困难。

等我写完插件才发现，官方开发了一个 chrome 插件来专门调试 tableau extensions。这是[文档](https://tableau.github.io/extensions-api/docs/trex_debugging.html)，这是[chrome插件下载地址](https://www.googleapis.com/download/storage/v1/b/chromium-browser-snapshots/o/Mac%2F352221%2Fchrome-mac.zip?generation=1443838516381000&alt=media)。
