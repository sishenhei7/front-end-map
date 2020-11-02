## 概述

今天我无意中看到一个库：[website-failure-check-box](https://github.com/Himself65/website-failure-check-box/blob/master/utils/index.js)，作用应该是检测网站的可用性，觉得挺有意思的，于是看了下它的源码，记录下来，供以后开发时参考，希望对其它人也有用。

### 核心代码

检测网站可用性的核心代码如下：

```
let exist = false
for (let i = 0; i < 3; i++) {
    // tip: check 3 times
    exist |= await axios.get(site).then(response => response.status !== 404).catch(error => signale.error(error))
}
```

可以看到，它是通过使用 axios 的 get 方法，然后判断返回的 status 是不是 404 来进行检测的。为了避免误差，检测了三次，并且用```|=```运算符进行汇总。

这个库是利用 github actions 进行定时自动检测的，github actions 我就不在这里介绍了，请自行了解。


