## 概述

今天我用**js给dom元素设置样式**，碰到了一些问题，记下来供以后开发时参考，相信对其他人也有用。

### 心得

1. js加上class: $dom.classList.add('some-class');
2. js去除class: $dom.classList.remove('some-class');
3. js设置行内样式: $dom.style.height = '10px';
4. js去除行内样式：$dom.style.height = '';

值得注意的是，**js去除行内样式是直接赋值一个空字符串即可**！！！
