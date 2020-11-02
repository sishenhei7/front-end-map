## 概述

Vue3的一个重大升级就是使用 proxy 来做数据劫持，我们来体验一下用 proxy 是怎么做数据劫持的，供以后工作时参考，相信对其它人也有用。

### Vue2.x的缺点

Vue2.x是使用```Object.defineProperty```来做数据劫持的，但是它有以下三个缺点：

```
1.不能劫持数组的变化，需要做特殊处理（通过劫持数组的push、splice等方法实现的）
2.必须深度遍历对象的每个属性
3.无法监听属性的新增和删除操作（通过Vue.set 和 Vue.delete实现的）
```

### 使用 proxy

1.劫持数组的变化

```js
const test = [];
const testProxy = new Proxy(test, {
    get(target, key) {
        if (key !== 'length') {
            console.log('get==================', target, key);
        }

        return target[key];
    },
    set(target, key, value) {
        if (key !== 'length') {
            console.log('set==================', target, key);
        }

        return Reflect.set(target, key, value);
    }
});

testProxy.push(1);
testProxy[0] = 2;
testProxy[1] = 'abc';
```

这里需要注意的是，这里使用push的时候，length属性会发生变化，所以上面过滤掉了length属性。（当然我这里做的比较简单，而 Vue3 源码里面是通过判断 push、indexOf、shift 等方法来分情况做的。）

2.必须深度遍历对象的每个属性

```js
const toProxy = new WeakMap();
const toRaw = new WeakMap();
const isObject = val => typeof val === 'object' && val !== null;
const proxyConfig = {
    get(target, key) {
        const res = Reflect.get(target, key);
        if (isObject(res)) {
            return reactive(res);
        }

        console.log('get==================', target, key);
        return res;
    },
    set(target, key, value) {
        console.log('set==================', target, key);
        return Reflect.set(target, key, value);
    }
}

function reactive(target) {
    const res = toProxy.get(target);

    if (res) {
        return res;
    }

    if (toRaw.get(target)) {
        return target;
    }

    const observed = new Proxy(target, proxyConfig);
    toProxy.set(target, observed);
    toRaw.set(observed, target);
    return observed;
}

const test = {
    a: 2,
    b: {
        c: 'abc',
    }
};
const testProxy = reactive(test);
console.log(testProxy.b.c);
testProxy.b.d = 3;
```

这里需要注意的是，当属性是深度嵌套的时候，只会触发 getter，并不会触发setter，所以需要对深度嵌套的属性进一步使用 proxy！

但是使用 proxy 得到的新属性又不能挂载到对象上面去，所以需要使用一个 weakmap 储存起来，同时也能加快查找速度。

3.监听属性的新增和删除操作

```
const test = {a:2};
const testProxy = new Proxy(test, {
    deleteProperty(target, key) {
        console.log('delete==================', target, key);
        delete target[key];
    },
});

delete testProxy.a;
```

### 总结

1.proxy做数据劫持还是有一些痛点的，比如对于数组的push、unshift等方法需要做兼容，对于对象的深层属性仍然需要通过遍历来重新proxy。

2.proxy的功能不止如此，它还能拦截更多的属性，比如 has、defineProperty、apply 等。
