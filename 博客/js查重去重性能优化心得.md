## 概述

今天产品反映有个5000条数据的页面的保存按钮很慢，查看代码看到是因为点击保存按钮之后，进行了查重操作，而查重操作是用2个for循环完成了，时间复杂度是O(n^2)。没办法，只能想办法优化一下了。

主要参考了这篇文章：[JavaScript 高性能数组去重](https://www.cnblogs.com/wisewrong/p/9642264.html)

### 源码

简单来说，这个页面的要求是**查找一个数组中的重复项，并且返回重复项的行号**。源码简化后如下：

```
    checkData(tableData) {
      // console.time('数组检查重复项时间');
      // 检查重复项，检查空值（全局）
      const repeatMidArr = [];
      const repeatArr = [];

      for (let i = 0; i < tableData.length; i += 1) {
        // 检查重复项
        for (let j = i + 1; j < tableData.length; j += 1) {
          const arr1 = tableData[i].condition;
          const arr2 = tableData[j].condition;
          if (arr1.length === arr2.length && JSON.stringify(arr1) === JSON.stringify(arr2)) {
            repeatMidArr.push(i + 1);
            repeatMidArr.push(j + 1);
          }
        }
      }

      // 给repeatMidArr去重
      repeatMidArr = repeatMidArr.sort();
      if (repeatMidArr.length <= 1) {
        repeatArr = repeatMidArr;
      } else {
        repeatArr.push(repeatMidArr[0]);
        for (let i = 1; i < repeatMidArr.length; i += 1) {
          if (repeatMidArr[i] !== repeatMidArr[i - 1]) repeatArr.push(repeatMidArr[i]);
        }
      }
      // console.timeEnd('数组检查重复项时间');

      if (repeatArr.length !== 0) {
        this.sendRepeatMsg(repeatArr);
        return true;
      }

      return false;
    },
```

注意：
1. 因为需要对一个数组查重，所以使用了**JSON.stringify**把数组转化为字符串简单处理。
2. 给纯数字数组利用**sort方法**去重。

### 优化

优化的**核心思想是算法中的hash表**，也就是字典。在js中可以利用对象的键值不重复这个特性来把对象变成一个hash表。简化后的代码如下：

```
    checkData(tableData) {
      // console.time('数组检查重复项时间');
      // 检查重复项，检查空值（全局）
      const repeatObj = {};
      let repeatMidArr = [];
      let repeatArr = [];

      for (let i = 0; i < tableData.length; i += 1) {
        // 检查重复项(优化方法)
        const itemCondition = JSON.stringify(tableData[i].condition);
        const index = repeatObj[itemCondition];
        if (!index) {
          repeatObj[itemCondition] = i + 1;
        } else {
          repeatMidArr.push(index);
          repeatMidArr.push(i + 1);
        }
      }

      // 给repeatMidArr去重
      repeatMidArr = repeatMidArr.sort();
      if (repeatMidArr.length <= 1) {
        repeatArr = repeatMidArr;
      } else {
        repeatArr.push(repeatMidArr[0]);
        for (let i = 1; i < repeatMidArr.length; i += 1) {
          if (repeatMidArr[i] !== repeatMidArr[i - 1]) repeatArr.push(repeatMidArr[i]);
        }
      }
      // console.timeEnd('数组检查重复项时间');

      if (repeatArr.length !== 0) {
        this.sendRepeatMsg(repeatArr);
        return true;
      }

      return false;
    },
```

代码很简单，这里就不细说了。这种方法既然都能用到查重并返回重复项中，当然也能够用到去重里面去。

### 结果

优化之后，在5000条数据下，点击保存按钮的响应时间从35秒缩短到了3秒，性能提升了**10倍**！！！

