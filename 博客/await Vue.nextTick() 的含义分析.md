## 概述

今天看别人的单元测试代码的时候碰到了一段代码 **await Vue.nextTick()**，初看起来不是很懂，后来通过查资料弄懂了，记录下来，供以后开发时参考，相信对其他人也有用。

### await Vue.nextTick()

我们都用过 Vue.nextTick，但是在用的时候会在里面加一个回调函数的，但是有人直接这么使用：

```
await Vue.nextTick();
```

这是为什么呢？使用场景又是什么呢？

### Vue.nextTick

要了解这段代码的含义，我们首先来看 Vue.nextTick() 如果不加回调函数会怎样？

通过查阅官方文档，可以知道，Vue.nextTick() 里面如果加了回调，则会**在下次 DOM 更新循环结束之后执行延迟回调**。如果在修改数据之后立即使用这个方法，则可以获取更新后的 DOM。如果没有提供回调且在支持 Promise 的环境中 则会**返回一个 Promise！！！**

所以 await Vue.nextTick() 相当于在 await 后面加了一个 Promise。

### await

await 后面加一个 Promise 又会怎样呢？

通过查阅资料，我们可以知道，await 后面必须跟 Promise，否则会报错；如果跟了 Promise，那么当执行到这里的时候，会**先返回**，等 Promise 返回后，**再继续执行下面的代码**。比如下面这段代码：

```
async function f1() {
  console.log('xxxx');
  await new Promise();
  console.log('tttt');
}
```

当执行到 await new Promise(); 时，会先返回，等 Promise resolve 之后再才执行下面的 console.log('tttt');

### 示例

下面是一个简单的示例：

```
function genPromise() {
  return new Promise(resolve => {
	console.log('await start');
    setTimeout(() => {
	    console.log('await end');
      resolve();
    }, 0);
  });
}

async function f1() {
  console.log('xxxx');
  await genPromise();
  console.log('should be after await end');
}

f1();
```

最后的打印结果是：

```
xxxx
await start
await end
should be after await end
```

所以 await Vue.nextTick() 就和这个类似，它会在等 DOM 更新之后再执行后面的代码，**其实就相当于把里面的代码拿出来写在后面了(仔细一想，这不就是 await 的常规用法吗？)**。
