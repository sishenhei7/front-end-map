## 概述

今天在项目中用到了 getBoundingClientRect 和 requestAnimFrame ，查了下它们的polyfill，记录下来，供以后开发时参考，相信对其他人也有用。

### getBoundingClientRect

getBoundingClientRect 的 polyfill如下所示：

``` js
getBoundingClientRect(element) {
  const rect = element.getBoundingClientRect();

  // whether the IE version is lower than 11
  const isIE = navigator.userAgent.indexOf('MSIE') !== -1;

  // fix ie document bounding top always 0 bug
  const rectTop = isIE && element.tagName === 'HTML'
    ? -element.scrollTop
    : rect.top;

  return {
    left: rect.left,
    top: rectTop,
    right: rect.right,
    bottom: rect.bottom,
    width: rect.right - rect.left,
    height: rect.bottom - rectTop,
  };
}
```

### requestAnimFrame

requestAnimFrame 的 polyfill如下所示：

``` js
polyfillRAF() {
  let requestAnimFrame = window.requestAnimationFrame
    || window.webkitRequestAnimationFrame
    || window.MozRequestAnimationFrame
    || window.msRequestAnimationFrame
    || window.ORequestAnimationFrame;

  const getNow = Date.now || (() => +new Date());

  let lastTime = getNow();

  if (!requestAnimFrame) {
    requestAnimFrame = (callback) => {
      // How long did it take to render?
      const deltaTime = getNow() - lastTime;
      const delay = Math.max(0, 1000 / 60 - deltaTime);

      return window.setTimeout(() => {
        lastTime = getNow();
        callback();
      }, delay);
    };
  }

  return requestAnimFrame;
}
```

值得注意的是，在移动端上requestAnimFrame可能会有性能问题，这个时候在移动端上建议不使用requestAnimFrame而使用setTimeout，代码如下：

``` js
polyfillRAF() {
  let requestAnimFrame = window.requestAnimationFrame
    || window.webkitRequestAnimationFrame
    || window.MozRequestAnimationFrame
    || window.msRequestAnimationFrame
    || window.ORequestAnimationFrame;

  const getNow = Date.now || (() => +new Date());

  let lastTime = getNow();

  // 判断移动端
  if (_isMobile || !requestAnimFrame) {
    requestAnimFrame = (callback) => {
      // How long did it take to render?
      const deltaTime = getNow() - lastTime;
      const delay = Math.max(0, 1000 / 60 - deltaTime);

      return window.setTimeout(() => {
        lastTime = getNow();
        callback();
      }, delay);
    };
  }

  return requestAnimFrame;
}
```
