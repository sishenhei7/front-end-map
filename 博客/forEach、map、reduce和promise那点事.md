## forEach、map、reduce和promise那点事

通过此文，您可以学到：

1. forEach、map、reduce 后面能不能带 async 函数？
2. 怎么实现多个 promise 同步执行，不管有没有抛出错误，都把结果收集起来？

### forEach 后面能不能带async函数？

首先我们来模拟一个异步函数：

```js
const fetch = (forceTrue) => new Promise((resolve, reject) => {
    if (Math.random() > 0.2 || forceTrue) {
        resolve('success');
    } else {
        reject('error');
    }
});
```

然后我们来试试使用 forEach 来执行多个async函数：

```js
const result = [];

[0,1,2].forEach(async () => {
    const value = await fetch(true);
    result.push(value);
});

console.log(result); // 返回 []
```

可以看到，我们预期使用 await 获取 fetch 的结果后存入 result 里面去，但是后面打印出来的却是**空数组**。

我们使用 for 试一试：

```js
const result = [];

for (let i = 0; i < 3; ++i) {
    const value = await fetch(true);
    result.push(value);
}

console.log(result); // 返回['success', 'success', 'success']
```

可以看到，使用 for 的时候**按预期返回**了。那么为什么用 forEach 就不行呢？我们看一看[mdn上面forEach的polyfill](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)源码：

```js
// Production steps of ECMA-262, Edition 5, 15.4.4.18
// Reference: http://es5.github.io/#x15.4.4.18
if (!Array.prototype.forEach) {
  Array.prototype.forEach = function(callback, thisArg) {

    var T, k;

    if (this == null) {
      throw new TypeError(' this is null or not defined');
    }

    // 1. Let O be the result of calling toObject() passing the
    // |this| value as the argument.
    var O = Object(this);

    // 2. Let lenValue be the result of calling the Get() internal
    // method of O with the argument "length".
    // 3. Let len be toUint32(lenValue).
    var len = O.length >>> 0;

    // 4. If isCallable(callback) is false, throw a TypeError exception.
    // See: http://es5.github.com/#x9.11
    if (typeof callback !== "function") {
      throw new TypeError(callback + ' is not a function');
    }

    // 5. If thisArg was supplied, let T be thisArg; else let
    // T be undefined.
    if (arguments.length > 1) {
      T = thisArg;
    }

    // 6. Let k be 0
    k = 0;

    // 7. Repeat, while k < len
    while (k < len) {

      var kValue;

      // a. Let Pk be ToString(k).
      //    This is implicit for LHS operands of the in operator
      // b. Let kPresent be the result of calling the HasProperty
      //    internal method of O with argument Pk.
      //    This step can be combined with c
      // c. If kPresent is true, then
      if (k in O) {

        // i. Let kValue be the result of calling the Get internal
        // method of O with argument Pk.
        kValue = O[k];

        // ii. Call the Call internal method of callback with T as
        // the this value and argument list containing kValue, k, and O.
        callback.call(T, kValue, k, O);
      }
      // d. Increase k by 1.
      k++;
    }
    // 8. return undefined
  };
}
```

可以看到，里面是通过 while 循环里面多次调用```callback.call(T, kValue, k, O);```来实现的，由于它前面没有 await，所以每次循环执行到这里的时候并没有等待它执行完，最后打印 result 的时候就是空数组了。

### map 和 reduce 后面能不能带async函数？

我们再来看一看 map 和 reduce 的 demo：

```js
// map
const mapResult = [0,1,2].map(async () => await fetch(true));
console.log(mapResult); // 返回 [Promise, Promise, Promise]

// reduce
const reduceResult = [0,1,2].reduce(async (accu) => {
    const value = await fetch(true);
    accu.push(value);
    console.log('accu=====', typeof accu);
    return accu;
}, []); // 报错 Uncaught (in promise) TypeError: accu.push is not a function
console.log(reduceResult);
```

可以看到，两个都没有正常返回，我们先来看一看[mdn上面map的polyfill](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)源码：

```js
// Production steps of ECMA-262, Edition 5, 15.4.4.19
// Reference: http://es5.github.io/#x15.4.4.19
if (!Array.prototype.map) {

  Array.prototype.map = function(callback/*, thisArg*/) {

    var T, A, k;

    if (this == null) {
      throw new TypeError('this is null or not defined');
    }

    // 1. Let O be the result of calling ToObject passing the |this|
    //    value as the argument.
    var O = Object(this);

    // 2. Let lenValue be the result of calling the Get internal
    //    method of O with the argument "length".
    // 3. Let len be ToUint32(lenValue).
    var len = O.length >>> 0;

    // 4. If IsCallable(callback) is false, throw a TypeError exception.
    // See: http://es5.github.com/#x9.11
    if (typeof callback !== 'function') {
      throw new TypeError(callback + ' is not a function');
    }

    // 5. If thisArg was supplied, let T be thisArg; else let T be undefined.
    if (arguments.length > 1) {
      T = arguments[1];
    }

    // 6. Let A be a new array created as if by the expression new Array(len)
    //    where Array is the standard built-in constructor with that name and
    //    len is the value of len.
    A = new Array(len);

    // 7. Let k be 0
    k = 0;

    // 8. Repeat, while k < len
    while (k < len) {

      var kValue, mappedValue;

      // a. Let Pk be ToString(k).
      //   This is implicit for LHS operands of the in operator
      // b. Let kPresent be the result of calling the HasProperty internal
      //    method of O with argument Pk.
      //   This step can be combined with c
      // c. If kPresent is true, then
      if (k in O) {

        // i. Let kValue be the result of calling the Get internal
        //    method of O with argument Pk.
        kValue = O[k];

        // ii. Let mappedValue be the result of calling the Call internal
        //     method of callback with T as the this value and argument
        //     list containing kValue, k, and O.
        mappedValue = callback.call(T, kValue, k, O);

        // iii. Call the DefineOwnProperty internal method of A with arguments
        // Pk, Property Descriptor
        // { Value: mappedValue,
        //   Writable: true,
        //   Enumerable: true,
        //   Configurable: true },
        // and false.

        // In browsers that support Object.defineProperty, use the following:
        // Object.defineProperty(A, k, {
        //   value: mappedValue,
        //   writable: true,
        //   enumerable: true,
        //   configurable: true
        // });

        // For best browser support, use the following:
        A[k] = mappedValue;
      }
      // d. Increase k by 1.
      k++;
    }

    // 9. return A
    return A;
  };
}
```

可以看到，源码里面**每次 while 循环把 callback 返回的结果赋值给 mappedValue**，而 callback 在调用的时候也没有使用 await，所以结果数组的每一个值都是一个 promise。

我们再看一看[mdn上面reduce的polyfill](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)源码：

```js
// Production steps of ECMA-262, Edition 5, 15.4.4.21
// Reference: http://es5.github.io/#x15.4.4.21
// https://tc39.github.io/ecma262/#sec-array.prototype.reduce
if (!Array.prototype.reduce) {
  Object.defineProperty(Array.prototype, 'reduce', {
    value: function(callback /*, initialValue*/) {
      if (this === null) {
        throw new TypeError( 'Array.prototype.reduce ' +
          'called on null or undefined' );
      }
      if (typeof callback !== 'function') {
        throw new TypeError( callback +
          ' is not a function');
      }

      // 1. Let O be ? ToObject(this value).
      var o = Object(this);

      // 2. Let len be ? ToLength(? Get(O, "length")).
      var len = o.length >>> 0;

      // Steps 3, 4, 5, 6, 7
      var k = 0;
      var value;

      if (arguments.length >= 2) {
        value = arguments[1];
      } else {
        while (k < len && !(k in o)) {
          k++;
        }

        // 3. If len is 0 and initialValue is not present,
        //    throw a TypeError exception.
        if (k >= len) {
          throw new TypeError( 'Reduce of empty array ' +
            'with no initial value' );
        }
        value = o[k++];
      }

      // 8. Repeat, while k < len
      while (k < len) {
        // a. Let Pk be ! ToString(k).
        // b. Let kPresent be ? HasProperty(O, Pk).
        // c. If kPresent is true, then
        //    i.  Let kValue be ? Get(O, Pk).
        //    ii. Let accumulator be ? Call(
        //          callbackfn, undefined,
        //          « accumulator, kValue, k, O »).
        if (k in o) {
          value = callback(value, o[k], k, o);
        }

        // d. Increase k by 1.
        k++;
      }

      // 9. Return accumulator.
      return value;
    }
  });
}
```

可以看到，源码里面**每次循环的时候也是把 callback 的值赋给累计器 accu，而 callback 是一个 async 函数**，所以这里 accu 其实**是一个 promise 而不是数组**，它没有数组方法，所以报错了。

### 解决方案

老实说，forEach、map、reduce、filter 这些方法本意都是**针对同步函数**的，不太适合异步的场景。在异步的场景，**建议使用 for 和 for of 方法**。

但是虽然他们都是针对同步函数的，还是有**一些 hack 方法**可以让它们对异步函数也生效，比如 reduce 的 hack 代码如下所示：

```js
(async function () {
    const fetch = (forceTrue) => new Promise((resolve, reject) => {
        if (Math.random() > 0.2 || forceTrue) {
            resolve('success');
        } else {
            reject('error');
        }
    });

    const reduceResult = await [0,1,2].reduce(async (accu) => {
        const value = await fetch(true);
        const resolvedAccu = await accu;
        resolvedAccu.push(value);
        return resolvedAccu;
    }, []);

    console.log('====', reduceResult);
})()
```

上面的代码有这些需要注意的地方：

1. 由于累计器 accu 是 async 回调函数的返回值 promise，所以我们**使用 await 让它得出结果**。
2. 由于最后的结果也是 promise，所以我们在前面加上了 await，但是**await 只能在 async 函数里面使用**，所以我们在匿名函数那里加上了 async。

### promise

上面让我想起了 promise 的一种使用场景：我们知道 promise.all 可以获得同步的 promise 结果，但是它有一个缺点，就是只要一个 reject 就直接返回了，不会等待其它 promise 了。那么怎么**实现多个 promise 同步执行，不管有没有抛出错误，都把结果收集起来**？

一般来说可以使用 for 循环解决，示例如下：

```js
Promise.myAll = function (promiseArr) {
    const len = promiseArr.length;
    const result = [];
    let count = 0;

    return new Promise((resolve, reject) => {
        for (let i = 0; i < len; ++i) {
            promiseArr[i].then((res) => {
                result[i] = res;
                ++count;

                if (count >= len) {
                    resolve(result);
                }
            }, (err) => {
                result[i] = err;
                ++count;

                if (count >= len) {
                    resolve(result);
                }
            });
        }
    });
}

// test
const fetch = (forceTrue) => new Promise((resolve, reject) => {
    if (Math.random() > 0.2 || forceTrue) {
        resolve('success');
    } else {
        reject('error');
    }
});

Promise.myAll([fetch(), fetch(), fetch()])
    .then(res => console.log(res));  // ["success", "success", "error"]
```

但是如果注意到 promise 的**then 方法和 catch 方法都会返回一个 promise**的话，就可以用下面的简单方法：

```js
Promise.myAll = function (promiseArr) {
    // 就这一行代码！
    return Promise.all(promiseArr.map(item => item.catch(err => err)));
}

// test
const fetch = (forceTrue) => new Promise((resolve, reject) => {
    if (Math.random() > 0.2 || forceTrue) {
        resolve('success');
    } else {
        reject('error');
    }
});

Promise.myAll([fetch(), fetch(), fetch()])
    .then(res => console.log(res)); // ["error", "error", "success"]
```

