## 概述

如果我们想要从父组件获取子组件的生命周期，除了在子组件相应的生命周期内部向父组件通信这种做法之外，还可以使用生命周期钩子。

### 示例代码

```
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld
      @hook:beforeCreate="console('child: beforeCreate')"
      @hook:created="console('child: created')"
      @hook:beforeMount="console('child: beforeMount')"
      @hook:mounted="console('child: mounted')"
      @hook:beforeUpdate="console('child: beforeUpdate')"
      @hook:updated="console('child: updated')"
      @hook:beforeDestroy="console('child: beforeDestroy')"
      @hook:destroyed="console('child: destroyed')"
      msg="Welcome to Your Vue.js App"
    />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  methods: {
    console(str) {
      console.log(str);
    },
  },
  beforeCreate() {
    console.log('parent: beforeCreate');
  },
  created() {
    console.log('parent: created');
  },
  beforeMount() {
    console.log('parent: beforeMount');
  },
  mounted() {
    console.log('parent: mounted');
  },
  beforeUpdate() {
    console.log('parent: beforeUpdate');
  },
  updated() {
    console.log('parent: updated');
  },
  beforeDestroy() {
    console.log('parent: beforeDestroy');
  },
  destroyed() {
    console.log('parent: destroyed');
  },
}
</script>
```

控制台打印如下：

```
parent: beforeCreate
parent: created
parent: beforeMount
child: beforeCreate
child: created
child: beforeMount
child: mounted
parent: mounted
parent: beforeUpdate
child: beforeUpdate
child: updated
parent: updated
parent: beforeDestroy
child: beforeDestroy
child: destroyed
parent: destroyed
```

### 源码位置

``` js
// init.js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}

// lifecycle.js
export function callHook (vm: Component, hook: string) {
  // #7573 disable dep collection when invoking lifecycle hooks
  pushTarget()
  const handlers = vm.$options[hook]
  const info = `${hook} hook`
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      invokeWithErrorHandling(handlers[i], vm, null, vm, info)
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  popTarget()
}
```

可以看到，每次触发钩子的时候是通过 callHook 进行触发的，而 callHook 里面会 emit 这个生命周期的钩子。

### _hasHookEvent

可以看到，前面会判断 _hasHookEvent ，这是干嘛的？源码如下：

```
// events.js
export function initEvents (vm: Component) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}

export function eventsMixin (Vue: Class<Component>) {
  const hookRE = /^hook:/
  Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {
    const vm: Component = this
    if (Array.isArray(event)) {
      for (let i = 0, l = event.length; i < l; i++) {
        vm.$on(event[i], fn)
      }
    } else {
      (vm._events[event] || (vm._events[event] = [])).push(fn)
      // optimize hook:event cost by using a boolean flag marked at registration
      // instead of a hash lookup
      if (hookRE.test(event)) {
        vm._hasHookEvent = true
      }
    }
    return vm
  }
  // ...
}
```

可以看到，在初始化事件的时候，会将 _hasHookEvent 设置为false，然后在```$on```方法里面判断是否有 hook 钩子，有的话就把 _hasHookEvent 设置为 true。我们在子组件上绑定了 hook 钩子，所以其内部会调用这个```$on```方法把 _hasHookEvent 设置为 true，从而发出生命周期钩子事件。

### 为什么会有这个设置，但又不写到文档里面去

我猜这个是遗留代码，目前已经没什么用了的原因。。。。
