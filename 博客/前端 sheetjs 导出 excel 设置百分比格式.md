## 概述

前端导出 excel 文件常用的库是 [sheetjs](https://github.com/SheetJS/sheetjs)。今天 PM 提了一个需求就是所有的数字都使用数字格式，特别是百分比也使用数字格式。我想了很久终于解决了，记录下来，供以后开发时参考，相信对其它人也有用。

### 问题

通过查看 [sheetjs的官方文档](https://github.com/SheetJS/sheetjs)，我们可以发现，它支持这几种导出格式：Boolean、Error、Number、Date、Text、Stub，里面没有百分比格式，怎么办呢？

方法就是使用格式化为 0.00% 的数字格式，其中后面2个0是自定义2个小数位，可以多加或者少加。

### 源码

Show me the code:

```js
const workbook = XLSX.utils.book_new();
const sheet = XLSX.utils.json_to_sheet(data);
Object.keys(sheet).forEach((item) => {
    const cell = sheet[item];
    const value = cell.v;
    if (cell.t === 's' && value.indexOf('%') > -1) {
        cell.z = '0.00%';
        cell.t = 'n';
        cell.v = Number(value.substring(0, value.length - 1)) / 100;
    }
});
XLSX.utils.book_append_sheet(workbook, sheet, 'Sheet1');
XLSX.writeFile(workbook, filename);
```

### 题外话

- 这个库真的很强大，还可以自定义很多其他的东西。
- 安利一下我自己写的 excel 导出库[download-excel](https://github.com/sishenhei7/download-excel)
