## 概述

以前开发项目的时候，总是分不清楚 preventDefault 和 stopPropagation，每次都是用 ```@click.stop```试一下，不能就用```@click.prevent```试一下。今天来好好总结一下这2个东西，记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：[preventDefault()、stopPropagation()、return false 之间的区别](https://www.cnblogs.com/dannyxie/p/5642727.html)

### 区别

preventDefault()是**阻止浏览器执行默认行为**，比如阻止浏览器的a标签**打开新窗口**，或者阻止浏览器的**默认滚动事件**。

stopPropagation()是**阻止事件冒泡**，比如如果我不想让子元素的点击事件冒泡到父元素。

注意：

1. return false 同时执行了上面2种行为，建议在大多数情况下不要使用return false。

2. stopImmediatePropagation 会停止绑定在元素上的剩余事件处理函数继续执行，一般我们很少用到。
