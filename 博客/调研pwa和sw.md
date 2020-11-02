## 概述

处于好奇，最近我调研了一下pwa和service worker，有些新的，记录下来，供以后开发时参考，相信对其他人也有用。pwa主要是通过service worker实现的，它主要包括**桌面图标，离线缓存和消息推送**这三个方面。

注意：开启service worker需要https环境，详细搭建流程可以看我上一篇博文。

service worker的离线缓存可以使用封装好的[sw-precache](https://github.com/GoogleChromeLabs/sw-precache)库实现，其它功能可以用[sw-toolbox](https://github.com/GoogleChromeLabs/sw-toolbox)库实现；另外还有一个新的库[workbox-sw](https://storage.googleapis.com/workbox-cdn/releases/3.3.0/workbox-sw.js)可以取代这2个库。

参考资料：

[Service Workers 和离线缓存](https://www.jianshu.com/p/b14d76eb594e)

[Web Push](http://thihara.github.io/Web-Push/)

[mdn Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)

[RE0：在Vue里用Service Worker来搞个中间层（React同理）（超详细）](https://www.colabug.com/3479278.html)

### 桌面图标

实现桌面图标需要项目中有service worker和manifest文件。

其中pc端不支持manifest文件，ios移动端也不支持，只有安卓移动端才支持manifest文件。

### 离线缓存

service worker有这样一条特性，就是它**会拦截所有的http请求**，所以在使用service worker的时候需谨慎。

并且，service worker在**pc和移动包括ios**上面都已经有良好的支持了。

另外，service worker还有离线缓存的api可以缓存数据，可以在拦截http请求后判断是否需要使用离线数据，也可以设置以什么方式使用离线数据。

### 消息推送

service worker其实在本地进程中开了一个**本地服务器**，然后可以通过这个服务器来做许多有意思的事情，比如消息推送。

消息推送的原理是，在service worker创建的时候就给远程推送服务器发送一个标识，并且监听推送事件，然后远程推送服务器在需要推送的时候就给标识列表的所有服务器发送推送信息，然后service worker就可以接收这些推送信息，利用h5的notification api显示推送信息。

### 其它

由于service worker在浏览器本地开了一个服务器（是在进程里面，可以不会因为浏览器关闭而关闭），所以它还能做另外一些**激动人心的事情**：

1. sw服务器中间层。一般来说，很多项目会使用nodejs搭建一个数据中间层处理数据，但是现在可以把这个中间层放到浏览器的sw里面。

2. 缓存自动更新，因为我们的数据都是通过api请求的，如果缓存这些数据就不能更新拿到最新数据了，但是通过sw就可以在需要更新的时候接收远程服务器的更新推送，然后拦截http请求，给有需要的接口重新发送http请求，给不需要的接口就用缓存的数据。

3. sw处理业务数据。大数据相关的项目，一般在接口方面需要先进行很多和业务数据相关的处理，如果在项目中处理的话会很繁琐，现在可以拿到sw里面处理。



