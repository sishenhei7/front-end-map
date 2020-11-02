## 概述

最近最近做项目的时候总会思考一些大的**应用设计模式相关**的问题，我把自己的思考记录下来，供以后开发时参考，相信对其他人也有用。

### 绑定变量

一般情况下，如果我们需要在组件中使用某个变量，会这么使用：

``` js
data() {
  return {
    myData: [],
  };
}
```

如果这个变量是**外部变量**，例如从外部文件引入的话，就会这么使用：

```js
import { provinces } from '@/util/consts';

export default {
  data() {
    return {
      myData: provices,
    };
  },
}
```

### 问题

但是如果这个变量是一个嵌套层级很深，数据量很大的对象的话，如果按照上面那样使用，vue 就会去**遍历这个变量的所有属性**，来监听这个变量的变化。非常的消耗性能，一个典型的例子是：

``` js
export default {
  data() {
    return {
      bannerBg: null,
    };
  },
  mounted() {
    this.loadScript('/js/three.min.js', () => {
      this.loadScript('/js/vanta.net.min.js', () => {
        this.bannerBg = window.VANTA.NET({
          el: '#bannerBg',
          color: 0x2197F3,
          backgroundColor: 0x071E31,
        });
      });
    });
  },
  beforeDestroy() {
    this.bannerBg.destroy();
  },
  methods: {
    loadScript(path, callback) {
      const script = document.createElement('script');
      script.src = path;
      script.language = 'JavaScript';
      script.onload = () => callback();
      document.body.appendChild(script);
    },
  },
}
```

上面的例子中，我们为了避免**内存泄漏**，在 beforeDestroy 生命周期里面进行**回收**，而为了获取回收的变量，我们把它绑定给了 this.bannerBg。

但是事实是，我们并不需要监听 this.bannerBg 这个变量，而这么绑定的结果是，这个 vue 组件在 mounted 的时候需要遍历 this.bannerBg 来增加 vue 的监听属性，**非常消耗性能**。

### 解决方案

所以，我们建议不把 bannerBg 放到 data() 里面去监听，而是**直接绑定给 this **就行了。优化后的代码如下：

``` js
export default {
  mounted() {
    this.loadScript('/js/three.min.js', () => {
      this.loadScript('/js/vanta.net.min.js', () => {
        this.bannerBg = window.VANTA.NET({
          el: '#bannerBg',
          color: 0x2197F3,
          backgroundColor: 0x071E31,
        });
      });
    });
  },
  beforeDestroy() {
    this.bannerBg.destroy();
  },
  methods: {
    loadScript(path, callback) {
      const script = document.createElement('script');
      script.src = path;
      script.language = 'JavaScript';
      script.onload = () => callback();
      document.body.appendChild(script);
    },
  },
}
```

如果这个变量不是过程中生成的，而是**初始化的时候生成**的，我们建议在 data() 方法里面这么做：

``` js
import { provinces } from '@/util/consts';

export default {
  data() {
    this.myData = provices;

    return {
      // 移到上面去了
      // myData: provices,
    };
  },
}
```
