## 概述

说到 vue 的响应式原理，我们都能很快答出数据劫持和发布者订阅者模式，通过 Object.defineProperty 来劫持 getter 和 setter，在 getter 的时候订阅依赖，在 setter 的时候发布响应执行依赖，从而达到响应式的目的。

但是如果深入一点，它是怎么收集、发布、管理依赖的呢？或者说，源码里面的 defineReactive、Dep、Watcher 之间有什么样的关系呢？

我通过自己写了一个简易的响应式系统弄懂了这些，记录下来，供以后工作时参考，相信对其他人也有用。

### 小型的响应系统

为了回答上面的问题，我打算写一个简易的的响应式系统，为了简便，这个系统只考虑没有嵌套的对象，代码如下：

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

class Watcher {
    constructor(cb) {
        this.getter = cb;
        this.deps = [];
        this.value = this.get();
    }

    get() {
        Dep.target = this;
        const value = this.getter();
        console.log('Dep.target', Dep.target);
        return value;
    }

    addDep(dep) {
        dep.addSub(this);
    }

    update() {
        const value = this.get();
    }
}

const obj = {};

defineReactive(obj, 'text', 'Hello World!');

const watcher = new Watcher(() => {
    document.querySelector('body').innerHTML = obj.text;
});
```

把上面的代码复制到浏览器的控制台运行，就可以看到浏览器里面出现了```Hello World```，然后我们继续在控制台输入```obj.text = 'Define Reactive'```，可以看到浏览器里面的```Hello World```就变成了```Define Reactive```。

### 结合 Vue 的生命周期

我们把上面的例子带入 Vue 的生命周期里面看：

1.首先我们知道，在```beforeCreate```阶段，Vue会进行各种初始化，比如事件、生命周期等，其实就等效于这段代码：

```js
const obj = {};
```

2.在```created```阶段，Vue会初始化状态：data、compute、props、methods等，那其实就等效于这段代码：

```js
defineReactive(obj, 'text', 'Hello World!');
```

3.在```beforeMount```阶段，Vue会对 vm 建立一个 watcher，当变动的时候，使用 update 进行更新，这就等效于这段代码：

```js
const watcher = new Watcher(() => {
    document.querySelector('body').innerHTML = obj.text;
});
```

### 总结

1.必须先用```defineReactive```设置为响应式的，然后才能在```watcher```里面实现依赖收集，并且实例化多个```watcher```都能被收集到。

2.总的来说，每一个 dep 实例其实就是一个订阅中心，它通过闭包存在，然后在通过 defineReactive 把数据转变为响应式之后，此时的数据没有任何变化，但是一旦实例化了一个 watcher，这个 watcher 如果引用了这个数据（getter），那么数据就会自动把这个 watcher 添加到自己的订阅中心，当改变这个数据的时候（setter），也会自动发布通知，达到更新订阅的 watcher 的目的。

3.vue2.x的源码的响应式系统除了上面的代码，还做了很多别的工作，比如观测嵌套对象、观测数组、处理已经存在的getter和setter、对 watcher 进行调度等等。
