## 概述

最近在开发 Vue 项目的时候遇到了内存泄漏问题，记录下来，供以后开发时参考，相信对其他人也有用。

### 背景

背景是需要用 three.min.js 和 vanta.net.min.js 给首页加上动画效果。

### 内存泄漏

我们的代码是这样的：

```
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
methods: {
  loadScript(path, callback) {
    const script = document.createElement('script');
    script.src = path;
    script.language = 'JavaScript';
    script.onload = () => callback();
    document.body.appendChild(script);
  },
},
```

这样就导致，在每次首页加载的时候，都会去请求 three.min.js 和 vanta.net.min.js 静态资源，并且初始化。然后由于组件销毁的时候并没有清除 window.VANTA.NET 方法施加的内存，所以导致了内存泄漏。具体表现是，切换首页和其它页，切换的次数多了的话，就会卡住。

那一般的方法是直接在 beforeDestroy 生命周期里面 执行 this.bannerBg.destroy() 就行了。但是一开始我并不知道 this.bannerBg 带有destroy 方法，所以走了下面的不少弯路。

### 检查内存泄漏

按照[官网的方法](https://cn.vuejs.org/v2/cookbook/avoiding-memory-leaks.html)，Mac 下打开 Chrome 任务管理器的方式是选择 Chrome 顶部导航 > 窗口 > 任务管理；在 Windows 上则是 Shift+Esc 快捷键。则可以打开 Chrome 的任务管理器，主要查看 Browser 和 GPU Process 的 内存占用空间。前者是查看程序执行所消耗的内存，后者是查看动画效果所消耗的内存。

### keep-alive

如果插件本身没有提供 destroy 方法的话，可以使用官方的替代方案：使用 keep-alive 组件，在内存中保留组件的状态。示例代码如下：

```
<keep-alive>
  <my-component v-if="show"></my-component>
</keep-alive>
```

但是我实际使用下来，并没有用，估计是因为 three.js 生成的 WebGL 动画导致的不是 Browser 的内存泄漏而是 GPU 的内存泄漏吧。

如果这种方法不行的话，估计就真没办法避免内存泄漏了，幸好 vanta.net 其实暴露了 destroy 方法！

我看了下和这个场景比较相似的[vue-particles](https://github.com/creotip/vue-particles)源码，它竟然没有使用 destroy 释放内存！！！

### 一个坑

在解决的过程中，我碰到了一个坑，就是我原本以为在 mounted 生命周期内的程序不会影响渲染，因为当执行到 mounted 生命周期的时候，Dom已经挂载好了。但是其实我错了，如果在 mounted 生命周期里面执行一段非常耗时的代码，浏览器是会一直白屏的，代码示例如下：

```
mounted() {
  let a = 1;
  while (a < 123456) {
    console.log('aaaaa', a);
  }
}
```

原因我暂时不知道。。。。。。。。
