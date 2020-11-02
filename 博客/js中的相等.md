## 概述

今天学习 jest，看文档的时候发现 jest 用到了 Object.is()，以前没有见过，所以记录下来，供以后开发时参考，相信对其他人也有用。

注意：Object.is的文档在[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is)

### Object.is

如果下列任何一项相同，则 Object.is(value1, value2) 返回 true：

- 两个值都是 undefined
- 两个值都是 null
- 两个值都是 true 或者都是 false
- 两个值是由相同个数的字符按照相同的顺序组成的字符串
- 两个值指向同一个对象
- 两个值都是数字并且
- 都是正零 +0
- 都是负零 -0
- 都是 NaN
- 都是除零和 NaN 外的其它同一个数字

### 相等

1. == 会做隐式转换，把两边的操作数的类型转化为相同。
2. === 不会做隐式转换，但是会认为 -0 和 +0 相等，并且认为 Number.NaN 不等于 NaN。
3. Object.is() 不会做隐式转换，但是认为-0 和 +0 不相等，并且Number.NaN 等于 NaN。

例子如下：

```
Object.is('foo', 'foo');     // true
Object.is(window, window);   // true

Object.is('foo', 'bar');     // false
Object.is([], []);           // false

var foo = { a: 1 };
var bar = { a: 1 };
Object.is(foo, foo);         // true
Object.is(foo, bar);         // false

Object.is(null, null);       // true

// 特例
Object.is(0, -0);            // false
Object.is(0, +0);            // true
Object.is(-0, -0);           // true
Object.is(NaN, 0/0);         // true
```

### Object.is的polyfill

```
if (!Object.is) {
  Object.is = function(x, y) {
    // SameValue algorithm
    if (x === y) { // Steps 1-5, 7-10
      // Steps 6.b-6.e: +0 != -0
      return x !== 0 || 1 / x === 1 / y;
    } else {
      // Step 6.a: NaN == NaN
      return x !== x && y !== y;
    }
  };
}
```

