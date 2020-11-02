## 概述
  项目中需要把一个DOM元素自动滚动到视野内，百思不得其解，最后再element库里面发现了这个方法，记录下来供以后开发时参考，相信对其他人也有用。
  参考资料：[element scroll-into-view.js](https://github.com/ElemeFE/element/blob/dev/src/utils/scroll-into-view.js)

### 代码

代码如下：

```
/* eslint-disable no-param-reassign */
export default function scrollIntoView(container, selected) {
  if (!selected) {
    container.scrollTop = 0;
    return;
  }

  const offsetParents = [];
  let pointer = selected.offsetParent;
  while (pointer && container !== pointer && container.contains(pointer)) {
    offsetParents.push(pointer);
    pointer = pointer.offsetParent;
  }
  const top = selected.offsetTop + offsetParents.reduce((prev, curr) => (prev + curr.offsetTop), 0);
  const bottom = top + selected.offsetHeight;
  const viewRectTop = container.scrollTop;
  const viewRectBottom = viewRectTop + container.clientHeight;

  if (top < viewRectTop) {
    container.scrollTop = top;
  } else if (bottom > viewRectBottom) {
    container.scrollTop = bottom - container.clientHeight;
  }
}
```