## 概述

说到 computed 和 watch 有什么不同，也许大多数人都知道：computed 是用现有数据生成一个新数据，并且能够被缓存；而 watch 是根据数据变化，执行一些回调函数，它有很多配置比如 deep、immediate 等。

大家也都知道，watch 只是源码里面 watcher 的一个实例，computed 属性也用到了 watcher，但是 computed 属性为什么能够相互依赖变化呢？明显 watcher 自己是做不到这一点的，因为 watcher 并不能 update 其它 watcher。我为了弄懂其中的原理根据 vue2.x 的源码写了一个简易的 computed 属性，供以后工作时参考，相信对其他人也有用。

部分代码来源于[Vue2.x是怎么收集依赖的](https://www.cnblogs.com/yangzhou33/p/13786683.html)

### 简易的 computed

为了简便，暂不考虑 computed 的 setter 的情况，我实现了一个简易的 computed，代码如下：

```js
function defineReactive(obj, key, val) {
    const dep = new Dep();

    Object.defineProperty(obj, key, {
        enumerable: true,
        configurable: true,
        get() {
            if (Dep.target) {
                dep.depend();
            }
            return val;
        },
        set(newVal) {
            val = newVal;
            dep.notify();
        }
    });
}

class Dep {
    constructor() {
        this.subs = [];
    }

    addSub(sub) {
        this.subs.push(sub);
    }

    removeSub() {
        const index = this.subs.indexOf(sub);
        if (index > -1) {
            this.subs.splice(index, 1);
        }
    }

    depend() {
        if (Dep.target) {
            Dep.target.addDep(this);
        }
    }

    notify() {
        const subs = this.subs.slice();
        for (let i = 0, l = subs.length; i < l; i++) {
            subs[i].update();
        }
    }
}

Dep.target = null;
const targetStack = []

function pushTarget (target) {
  targetStack.push(target)
  Dep.target = target
}

function popTarget () {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}

class Watcher {
    constructor(cb, dirty = false) {
        this.getter = cb;
        this.deps = [];
        this.newDeps = [];
        this.value = this.get();
        this.dirty = dirty;
    }

    get() {
        pushTarget(this);
        const value = this.getter();
        popTarget(this);
        this.deps = [...this.newDeps];
        this.newDeps = [];
        return value;
    }

    addDep(dep) {
        this.newDeps.push(dep);
        dep.addSub(this);
    }

    update() {
        this.dirty = true;
        this.value = this.get();
    }

    evaluate() {
        this.value = this.get();
        this.dirty = false;
    }

    depend() {
        let i = this.deps.length;

        while (i--) {
            this.deps[i].depend();
        }
    }
}

const obj = {};
defineReactive(obj, 'text', 'Hello World!');

const vm = {};
const computed = {
    text1() {
        return `${obj.text}-text1`;
    },
    text2() {
        return `${vm.text1}-text2`;
    }
};

function createComputedGetter(key) {
    return function computedGetter() {
        const watcher = vm.computedWatchers[key];

        if (watcher) {
            if (watcher.dirty) {
                watcher.evaluate();
            }

            if (Dep.target) {
                watcher.depend();
            }

            return watcher.value;
        }
    }
}

function defineCompute(target) {
    const watchers = vm.computedWatchers = Object.create(null);

    for (key in target) {
        const cb = target[key];
        watchers[key] = new Watcher(cb, true);

        //defineComputed
        Object.defineProperty(vm, key, {
            get: createComputedGetter(key),
            set(a) {
                return a;
            }
        });
    }
}

defineCompute(computed);

const watcher = new Watcher(() => {
    document.querySelector('body').innerHTML = vm.text2;
});
```

把上面的代码复制到浏览器的控制台运行，就可以看到浏览器里面出现了```Hello World-text1-text2```，然后我们继续在控制台输入```obj.text = 'Define Reactive'```，可以看到浏览器里面的```Hello World-text1-text2```就变成了```Define Reactive-text1-text2```。

显然，由于我们改变了```obj.text```的值，然后自动的导致了```vm.text1```和```vm.text2```的值发生了响应式变化。

而其中的原理是，假如计算属性 A 依赖计算属性 B，而计算属性 B 又依赖响应式数据 C，那么最一开始先把计算属性 AB 都转化为 watcher，然后在把计算属性 AB 挂载到 vm 上面的时候，插入了一段 getter，而计算属性 B 的这个 getter 在这个计算属性 B 被读取的时候会把计算属性 A 的 watcher 添加到响应式数据 C 的依赖里面，所以响应式数据 C 在改变的时候会先后导致计算属性 B 和 A 执行 update，从而发生改变。

而其中关键的那段代码就是这段：

``` js
function createComputedGetter(key) {
    return function computedGetter() {
        const watcher = vm.computedWatchers[key];

        if (watcher) {
            if (watcher.dirty) {
                watcher.evaluate();
            }

            // 这里非常关键
            if (Dep.target) {
                watcher.depend();
            }

            return watcher.value;
        }
    }
}
```

为什么在计算属性 B 的 getter 函数里面会添加计算属性 A 的 watcher 呢？这是因为计算属性 B 在求值完成后，会自动把```Dep.target```出栈，从而暴露出计算属性 A 的 watcher。代码如下：

```js
class Watcher {
    get() {
        // 这里把自己的 watcher 入栈
        pushTarget(this);
        const value = this.getter();
        // 这里把自己的 watcher 出栈
        popTarget(this);
        this.deps = [...this.newDeps];
        this.newDeps = [];
        return value;
    }
}
```

这就是 pushTarget 和 popTarget 调度 watchers 的美丽之处~~

### 其它

需要注意以下两点：

1.在给计算属性生成 getter 的时候，不能直接使用 Object.defineProperty，而是使用闭包把 key 值储存了起来。

2.为什么不直接使用 defineReactive 把计算属性变成响应式的。因为当把计算属性用 setter 挂载到 vm 上面的时候，计算属性这里确实变成了一个具体的值，但是如果使用 defineReactive 把计算属性变成响应式的话，计算属性会执行自己的依赖，从而和响应式数据的依赖重复了。其实这也是把非数据变成响应式的一种方法。
