## 概述

项目中碰到一个问题，就是在ios机上，用iframe内嵌的网页**有时需要登录，有时候又不需要登录**。查找了半天，终于发现是[ios的跨站脚本限制](https://webkit.org/blog/7675/intelligent-tracking-prevention/)导致的。这里就来介绍下**跨站脚本限制**，供以后开发时参考，相信对其他人也有用。

参考资料：[Intelligent Tracking Prevention](https://webkit.org/blog/7675/intelligent-tracking-prevention/)

### 跨站追踪和第三方cookie

假设，一个用户同时浏览example-products.com和example-recipies.com，而这2个网站都加载了example-tracker.com的内容，并且example-tracker.com能在用户的浏览器里面设置cookie的话，那么example-tracker.com能够知道用户在example-products.com和example-recipies.com所干的事。这就是**跨站追踪**，而example-recipies.com设置的cookie被称为**第三方cookie**。

### ios的应对策略

ios通过机器学习的方法，利用用户最近30天的行为数据，智能的分析哪些第三方cookie需要阻止，哪些不需要阻止。

当然，用户可以自行在ios浏览器里面**取消**跨站脚本限制。

### 对前端来说意味着什么

这就表示，在ios浏览器里面，不能用iframe等方式注入第三方cookie；同时也表示，如果iframe里面的网址是跨域的话，就不能获得里面网址的cookie，ios浏览器会阻止这种行为。

所以**前端不能通过第三方的形式注入cookie，也不能通过第三方的形式获取cookie，也就不能自动登录了**。

### 注意

有以下2点需要注意：

1. 无论是safari还是ios上的chrome抑或是ios上的微信浏览器，都会有这个跨站脚本限制。

2. 注意是**第三方**才会限制，也就是说，如果iframe的一级域名和外面的相同，就不会限制了。

所以这个问题只有一个**解决方案**，就是把iframe里面内容的一级域名弄的和外面的相同就行了！
