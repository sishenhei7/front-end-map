## vuex和普通的event-bus有什么不同

我们都知道，vuex 的底层实现原理其实就是 event-bus，那么它和普通的 event-bus 有什么不同呢？我们通过简单的源码一步步实现来搞懂这个问题。

参考资料：[手写Vuex核心原理](https://github.com/Candy-Bullet/howToBuildMyVuex)

### event-bus

首先一个普通的 event-bus 是这样的：

```js
// main.js
Vue.prototype.$bus = new Vue();

// 组件中
this.$bus.$on('console', (text) => {
    console.log(text);
});

// 组件中
this.$bus.$emit('console', 'hello world');
```

它是通过 Vue 的```$on```和```$emit``` api 来传递消息的。

### vuex 的响应式数据

而 vuex 的数据是**响应式**的，那么我们首先实现这种响应式数据：

```js
class store {
    constructor(options) {
        this.vm = new Vue({
            data: {
                state: options.state
            },
        });
    }

    get state() {
        return this.vm.state;
    }
}
```

注意，上面的data**不是一个函数**，因为这里我们只会实例化一次。然后我们通过添加一个 state 的 getter 方法来暴露内部的 event-bus 的 state 属性。

那怎么实现响应式的呢？因为在实例化 vm 的时候，Vue 会自己使用 defineReactive 把 data 变为响应式数据，从而会收集有关它的依赖，然后在自己变动的时候，通知依赖更新。


### 加上 getters

vuex支持加上 getters，怎么加呢？直接初始化一个 getters 属性即可：

```js
class store {
    constructor(options) {
        this.vm = new Vue({
            data: {
                state: options.state
            },
        });

        const getters = options.getter || {};
        this.getters = {};
        Object.keys(getters).forEach((key) => {
            Object.defineProperty(this.getters, key, () => {
                get: () => getters[key](this.state)
            })
        });
    }

    get state() {
        return this.vm.state;
    }
}
```

原理就是**添加一个 getters 属性**，然后遍历 getters 并绑定到它的各个属性的 getter 上面去即可。

### 加上 mutations

类似的，我们可以**添加一个 mutations 属性**来保存 mutations，然后实现一个 commit 方法，在调用 commit 方法的时候去 mutations 里面找，然后调用相应函数即可：

```js
class store {
    constructor(options) {
        this.vm = new Vue({
            data: {
                state: options.state
            },
        });

        const getters = options.getters || {};
        this.getters = {};
        Object.keys(getters).forEach((key) => {
            Object.defineProperty(this.getters, key, () => {
                get: () => getters[key](this.state)
            })
        });

        const mutations = options.mutations || {};
        this.mutations = {};
        Object.keys(getters).forEach((key) => {
            this.mutations[key] = (args) => mutations[key](state, args);
        });
    }

    get state() {
        return this.vm.state;
    }

    commit(name, args) {
        this.mutations[name](args);
    }
}
```

### 加上 actions

类似的，我们可以**添加一个 actions 属性**来保存 actions，然后实现一个 dispatch 方法，在调用 dispatch 方法的时候去 actions 里面找，然后调用相应函数即可。不过稍有不同的是，actions 里面的方法和 mutations 里面的方法的**第一个参数不同**，它要传整个 store 进去，因为不仅要获得 state，还需要在里面调用 commit 方法：

```js
class store {
    constructor(options) {
        this.vm = new Vue({
            data: {
                state: options.state
            },
        });

        const getters = options.getters || {};
        this.getters = {};
        Object.keys(getters).forEach((key) => {
            Object.defineProperty(this.getters, key, () => {
                get: () => getters[key](this.state)
            })
        });

        const mutations = options.mutations || {};
        this.mutations = {};
        Object.keys(mutations).forEach((key) => {
            this.mutations[key] = (args) => mutations[key](state, args);
        });

        const actions = options.actions || {};
        this.actions = {};
        Object.keys(actions).forEach((key) => {
            this.actions[key] = (args) => actions[key](this, args);
        });
    }

    get state() {
        return this.vm.state;
    }

    commit(name, args) {
        this.mutations[name](args);
    }

    dispatch(name, args) {
        this.actions[name](args);
    }
}
```

### 其它

后面install流程就和其它插件的install流程差不多一样了，唯一需要注意的是插入到 beforeCreate 钩子里面的代码，它是**通过获取父级的 store 来获取全局 store 的**。

可以看到，vuex 和传统的 event-bus 的不同点除了 vuex 实现了**更加友好的响应式状态**之外，还禁止了 vuex 里面数据的直接修改，大大增强了**信任度**（有点像promise的status），通过增加 mutations 和 actions 这种“中间层”，它能**更好的控制中间的变化**，比如实现时间旅行、状态回退和状态保存的功能。

另外，需要说明的是，mutation 和 action 的不同点**不仅在于 mutation 里面只能写同步代码，action 里面只能写异步代码**，还在于 mutation 里面的方法的**第一个参数**是**state**（因为只需要修改 state 就可以了），而 action 里面的方法的第一个参数是**store**（因为还需要调用 commit 方法）






