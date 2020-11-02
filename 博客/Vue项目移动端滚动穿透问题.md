## 概述

今天在做 Vue 移动端项目的时候遇到了**滚动穿透问题**，在网上查资料后，选取了我觉得最好的方法，记录下来供以后开发时参考，相信对其他人也有用。

### 上层无需滚动

如果上层无需滚动的话，**直接屏蔽上层的 touchmove 事件**即可。示例如下：

``` html
<div @touchmove.prevent>
我是里面的内容
</div>
```

### 上层需要滚动

如果上层需要滚动的话，那么固定的时候先获取 body 的滑动距离，然后用 fixed 固定，**用 top 模拟滚动距离**；不固定的时候用获取 top 的值，然后让 body **滚动到之前的地方**即可。示例如下：

```
<template>
  <div @click="handleHambergerClick"></div>
</template>
<script>
export default {
  name: 'BaseHeaderMobile',
  data() {
    return {
      isHeaderVisible: false,
    };
  },
  methods: {
    handleHambergerClick() {
      // hack: 滑动穿透问题
      if (!this.isHeaderVisible) {
        this.lockBody();
      } else {
        this.resetBody();
      }

      this.isHeaderVisible = !this.isHeaderVisible;
    },
    lockBody() {
      const { body } = document;
      const scrollTop = document.body.scrollTop || document.documentElement.scrollTop;
      body.style.position = 'fixed';
      body.style.width = '100%';
      body.style.top = `-${scrollTop}px`;
    },
    resetBody() {
      const { body } = document;
      const { top } = body.style;
      body.style.position = '';
      body.style.width = '';
      body.style.top = '';
      document.body.scrollTop = -parseInt(top, 10);
      document.documentElement.scrollTop = -parseInt(top, 10);
    },
  },
};
</script>
```

