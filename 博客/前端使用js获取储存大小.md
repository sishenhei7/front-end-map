## 前端使用 js 获取储存大小

前端**使用 js 获取储存大小**是一个很常见的需求，我们在这里整理一下各种使用场景。通过本文，您可以学到：

1. 怎么使用 js 获取用户上传文件的大小
2. 怎么使用 js 获取 cookie 或 session 的大小

### 怎么使用 js 获取用户上传文件的大小

主要原理是，上传文件的 input 里面可以拿到文件对象，然后就可以**直接从这个对象上面的 size 属性上**获取文件大小，如果需要兼容 ie 的话，需要额外处理：

```js
let target = document.getElementById('file');

if (!target.files) {
    var filePath = target.value;
    var fileSystem = new ActiveXObject("Scripting.FileSystemObject");
    var file = fileSystem.GetFile(filePath);
    fileSize = file.Size;
} else {
    fileSize = target.files[0].size;
}
```

### 怎么使用 js 获取 cookie 或 session 的大小

根据[JavaScript 数据类型和数据结构](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)可以知道：

1. 数字在 js 中是以 64 位储存的，所以每个数字类型占 8 字节
2. 字符串的每个字母在 js 中是以 16 位储存的，所以字符串的每个字母占 2 字节
3. 对于对象等引用类型，由于大部分是引用，所以主要由它内部的变量决定
4. 对于 TypedArray objects，储存大小通过查表获得

所以比如说我们要获取某个 sessionStorage 的大小，因为 sessionStorage 里面储存的都是字符串，所以我们**计算这个字符串所占的储存空间即可**。源码如下：

```js
function getSessionStorageSize(key) {
    const item = sessionStorage.getItem(key);
    let size = JSON.stringify(item).length * 2;
    const arr = ['bytes', 'KB', 'MB', 'GB', 'TB'];
    let sizeUnit = 0;

    while(size > 1024) {
        size /= 1024;
        ++sizeUnit;
    }

    return `${key}的大小为：${size.toFixed(2)}${arr[sizeUnit]}`;
}
```
