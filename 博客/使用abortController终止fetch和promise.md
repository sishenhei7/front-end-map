## 使用 abortController 终止 fetch 和 promise

在使用 fetch 和 promise 的时候，中途终止它们是一个很常见的需求，我们来看一看怎么实现。通过本文您可以学到：

1. 怎么在外面终止 xhr 请求
2. abortController 是什么
3. 怎么使用 abortController 终止 fetch 请求
4. 怎么使用 abortController 终止 promise 请求

### 怎么在外部终止 xhr 请求

我们先来回顾一下怎么在外部终止 xhr 请求:

```js
let xhr = new XMLHttpRequest();
xhr.onload = (res) => console.log(res);
xhr.onerror = (err) => console.log(err);
xhr.onabort = () => console.log('aborted!');
xhr.open('get', 'https://slowmo.glitch.me/5000');
xhr.send();

setTimeout(() => {
    xhr.abort();
}, 200);
```

### AbortController

参考资料：[abortController接口](https://developer.mozilla.org/zh-CN/docs/Web/API/FetchController)

AbortController 接口表示一个控制器对象，允许你根据需要中止一个或多个 Web 请求。目前这个接口的**兼容性是除了IE已经能够兼容所有主流浏览器**了。示例如下：

```js
const controller = new AbortController();
const signal = controller.signal;

signal.addEventListener('abort', () => {
    console.log('aborted!');
});

controller.abort();
```

### 中止 fetch 请求

fetch 方法的第二个参数可以接收 signal 参数，当被中止时，会 reject 一个名字为 AbortError 的 error 并被 catch 捕捉到，示例如下：

```js
const controller = new AbortController();
const signal = controller.signal;

fetch('https://slowmo.glitch.me/5000', { signal })
    .then(res => res.json())
    .then(res => console.log(res))
    .catch((err) => {
        if (err.name === 'AbortError') {
            console.log('aborted');
        } else {
            console.log('error');
        }
    });

setTimeout(() => {
    controller.abort();
}, 200);
```

### 中止 promise

其实**不需要 AbortController**也可以实现手动中止 promise，还是用那套劫持的方法。示例如下：

```js
class MyPromise {
    constructor(executor) {
        let abort = null;
        let p = new Promise((resovle, reject)=>{
            executor(resovle, reject);
            abort = err => reject(err);
        })

        p.abort = abort;
        return p;
    }
}

// test
let test = new MyPromise((resolve) => {
    setTimeout(() => resolve(1), 200);
});

// 这里不能直接把 then 和 catch 加到上面的末尾去
test.then(res => console.log(res))
.catch(err => console.log(err));

test.abort('aborted!');
```

上面有一些不完美，就是在初始化 test 的时候，**后面不能带上 then 或 catch 方法**，因为这些方法返回的是一个 Promise 实例而不是 MyPromise 实例。

使用 AbortController 可以解决这个问题，示例如下：

```js
class MyPromise {
    constructor(executor, { signal }) {
        return new Promise((resolve, reject)=>{
            executor(resolve, reject);

            if (signal) {
                signal.addEventListener('abort', () => {
                    reject('aborted!');
                });
            }
        })
    }
}

// test
const controller = new AbortController();
const signal = controller.signal;

let test = new MyPromise((resolve) => {
    setTimeout(() => resolve(1), 200);
}, { signal })
.then(res => console.log(res))
.catch(err => console.log(err));

controller.abort('aborted!');
```

上面两种方案看起来很美好，但是其实并不完美，因为 then 和 catch 方法**返回的是一个 Promise 实例而不是 MyPromise 实例**，所以目前这个 abort 方法**不能中断 then 或 catch 里面的内容**，如果要修正的话，需要再继续改写 then 和 catch 方法。



