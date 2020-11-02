## 深入理解class和装饰器

class 的出现大大简化了 javascript 中类的写法，而装饰器又是 class 里面非常实用的功能，但是老实说，它们都是**语法糖**，并没有引入新的功能，那它们的原理是怎样的呢？本文来一一探究。通过本文，您可以学到：

1. class 语法糖的原理是什么？
2. super 的原理是什么？有什么注意事项？
3. 装饰器的原理是什么？
4. vue-class-component 是怎么实现 vue 的 class 写法的？
5. vue-property-decorator 是怎么实现 watch 装饰器的？

### class 语法糖

我们首先用 class 的写法写一个 demo:

```js
class A {
  constructor(name) {
    this.name = name;
  }

  say() {
    console.log(this.name);
  }

  static move() {
    console.log('move');
  }
}

class B extends A {
  constructor() {
    this.a = 1;
    super();
    this.b = 2;
  }

  hello() {
    console.log('hello');
  }

  static go() {
    console.log('go');
  }
}
```

通过分析 babel 打包后的代码，其实可以简化成下面这样：

```js
var A = /*#__PURE__*/function () {
  function A(name) {
    this.name = name
  }

  A.prototype.say = function say() {
    console.log(this.name)
  }

  A.move = function move() {
    console.log('move')
  }

  return A
}()

var B = /*#__PURE__*/function (_A) {
  var _super = function _createSuperInternal() {
    return _A.apply(this, arguments) || this
  }

  function B() {
    var _this;

    // _this.a = 1; // 这里会出现 _this 未定义，导致报错，所以不能在 super 之前绑定实例属性
    _this = _super.call(this)
    console.log(_this)
    _this.b = 2;
    return _this;
  }

  // 这里其实等价于：
  // B.prototype = Object.create(_A.prototype)
  // B.prototype.constructor = B
  B.prototype = Object.create(_A.prototype, {
    constructor: {
      value: B,
      writable: true,
      configurable: true
    }
  })

  B.prototype.hello = function hello() {
    console.log('hello');
  }

  B.go = function go() {
    console.log('go');
  }

  return B;
}(A);
```

可以看到：

1. class 语法糖原理其实就是**使用 constructor 作为构造函数**，然后**在构造函数的 prototype 上面绑定实例方法**，并且**直接在构造函数上面绑定静态方法**。
2. class 的继承就是**组合继承**的形式。

### 关于 super

我们从上面可以看到，super 其实是内部创建的一个方法，它使用构造函数继承的方法来继承实例属性。但由于 _this 其实是由 super 返回的，所以如果在 super 之前绑定实例属性的话，_this 还未定义，导致报错。所以**在 super 之前不能绑定实例属性**。

那这里的 _this 为什么不直接用自己的 this 呢？

我们来思考这样一种场景，就是**构造函数里面有返回值**：

```js
class C {
  constructor() {
    return {a:2}
  }
}
```

上面的类会被编译成：

```js
var C = /*#__PURE__*/function () {
  function C(name) {
    return {a:2}
  }

  return C
}()
```

我们在实例化这个类的时候，其实得到的是构造函数的返回值，即```{a:2}```这个对象。所以如果父类是这种**返回对象的形式**的话，子类在继承的时候，就必须在这个**返回值**上面绑定实例属性，而不是在**自己的this**上面绑定实例属性。

### 装饰器

在 vue 中，我们一般使用[vue-class-component](https://github.com/vuejs/vue-class-component)来把 vue 里面组件的写法转变为类形式的，写法如下：

```vue
<template>
  <div>{{ message }}</div>
</template>

<script type='ts'>
import { Vue, Component, Watch } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
  // Declared as component data
  message = 'Hello World!'

  @Watch('visible')
  onVisibleChanged(newValue: any) {
    this.$emit('input', newValue);
  }
}
</script>
```

那么它是怎么实现的呢？主要分为 2 步：

1. 在打包的时候会把装饰器打包成原始代码
2. component 装饰器会对组件类做一些处理

### 装饰器是怎么打包的

装饰器是一个函数，它接收的三个参数：

1. target（对象的 prototype，如果是类装饰器的话，就只有这一个参数）
2. key（当前的方法名或属性名）
3. descriptor（就是用于 defineProperty 的 config）

我们经常使用的 class 装饰器有 2 种（存在的一共有4种，我们只讨论常见的这 2 种）：

1. 放在 class 上的**类装饰器**
2. 放在方法上的**方法装饰器**

首先我们来看一段示例代码：

```js
function show(target, key, descriptor) {
  console.log(target);
  console.log(key);
  console.log(descriptor);
}

// 类装饰器
@show
class A {
  constructor(name) {
    this.name = name;
  }

  // 方法装饰器
  @show
  say() {
    console.log(this.name);
  }
}
```

打包之后的简化代码如下：

```js
function _applyDecoratedDescriptor(target, property, decorators, descriptor) {
  var desc = {};

  Object.keys(descriptor).forEach(function (key) {
    desc[key] = descriptor[key];
  });

  desc = decorators.slice().reverse().reduce(function (desc, decorator) {
    return decorator(target, property, desc) || desc;
  }, desc);

  return desc;
}

function show(target, key, descriptor) {
  console.log(target);
  console.log(key);
  console.log(descriptor);
}

var A = show(_class = (_class2 = /*#__PURE__*/ function () {
  function A(name) {
    this.name = name;
  }

  A.prototype.say = function say() {
    console.log(this.name);
  }

  return A;
}(), (_applyDecoratedDescriptor(
  _class2.prototype,
  "say",
  [show],
  Object.getOwnPropertyDescriptor(_class2.prototype, "say")
  )
), _class2)) || _class;
```

可以看到：

1. 对于类装饰器，只接收了 _class 参数，而```_class =```后面的括号里的三个值其实是一种顺序写法，最终返回的是括号里面的最后那个值也就是 _class2。
2. 对于方法装饰器，会被放到一个数组里面去，然后调用 _applyDecoratedDescriptor 对被装饰的方法**顺序执行**各个装饰器（谁在上面谁先执行）。
3. _applyDecoratedDescriptor 会**收集 prototype、method key 和 property descriptor**，然后传给装饰器进行执行。
4. 这里注意一下类装饰器和方法装饰器的执行顺序：**先执行方法装饰器再执行类装饰器**。

到这里就很清晰了，装饰器其实并不是什么黑魔法，只是**在编译的时候依次给类或者对象执行的函数**罢了。

### vue-class-component 是怎么实现 vue 的 class 写法的？

[vue-class-component](https://github.com/vuejs/vue-class-component)是通过 @component 装饰器来实现 vue 的 class 写法的，源码如下：

```js
// index.ts
function Component (options: ComponentOptions<Vue> | VueClass<Vue>): any {
  // 要装饰的类其实是函数类型，所以会从这里进入
  if (typeof options === 'function') {
    return componentFactory(options)
  }
  // 如果传入一个对象的话，就返回一个装饰器函数
  return function (Component: VueClass<Vue>) {
    return componentFactory(Component, options)
  }
}
```

它对传入的参数做了一层适配：

1. 如果传入的是函数类型，则**证明是一个类**，然后直接返回装饰器。（所以能够支持```@component()```这种不加参数的写法）
2. 如果传入的不是函数类型，则**证明是一个配置**，然后返回装饰器函数。（所以能够支持```@component(config)```这种加参数的写法）

最终，它是通过调用 componentFactory 来进行装饰的，它的源码如下：

```js
export const $internalHooks = [
  'data',
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeDestroy',
  'destroyed',
  'beforeUpdate',
  'updated',
  'activated',
  'deactivated',
  'render',
  'errorCaptured', // 2.5
  'serverPrefetch' // 2.6
]

export function componentFactory (
  Component: VueClass<Vue>,
  options: ComponentOptions<Vue> = {}
): VueClass<Vue> {
  options.name = options.name || (Component as any)._componentTag || (Component as any).name
  // prototype props.
  const proto = Component.prototype
  Object.getOwnPropertyNames(proto).forEach(function (key) {
    if (key === 'constructor') {
      return
    }

    // 加上生命周期函数、钩子函数
    if ($internalHooks.indexOf(key) > -1) {
      options[key] = proto[key]
      return
    }
    const descriptor = Object.getOwnPropertyDescriptor(proto, key)!
    if (descriptor.value !== void 0) {
      // 加上 methods
      if (typeof descriptor.value === 'function') {
        (options.methods || (options.methods = {}))[key] = descriptor.value
      } else {
        // 使用 mixins 的形式加上 data
        (options.mixins || (options.mixins = [])).push({
          data (this: Vue) {
            return { [key]: descriptor.value }
          }
        })
      }
    } else if (descriptor.get || descriptor.set) {
      // 加上 computed
      (options.computed || (options.computed = {}))[key] = {
        get: descriptor.get,
        set: descriptor.set
      }
    }
  })

  // 收集 constructor 上的 data
  ;(options.mixins || (options.mixins = [])).push({
    data (this: Vue) {
      return collectDataFromConstructor(this, Component)
    }
  })

  // 处理其它装饰器（方法装饰器、属性装饰器等）
  const decorators = (Component as DecoratedClass).__decorators__
  if (decorators) {
    decorators.forEach(fn => fn(options))
    delete (Component as DecoratedClass).__decorators__
  }

  // 初始化 super 里面的实例属性
  const superProto = Object.getPrototypeOf(Component.prototype)
  const Super = superProto instanceof Vue
    ? superProto.constructor as VueClass<Vue>
    : Vue
  const Extended = Super.extend(options)

  // 处理子类和父类的静态方法
  forwardStaticMembers(Extended, Component, Super)

  // 复制使用 reflect 声明的属性
  if (reflectionIsSupported()) {
    copyReflectionMetadata(Extended, Component)
  }

  return Extended
}
```

总的来说，这段代码做了如下工作。其实就是**筛选出相应的属性和方法按 options 的形式进行组装罢了**。

1. 保存一个内部钩子列表，筛选出生命周期钩子、路由钩子等作为相关的方法。（所以我们如果要加入路由钩子的话，首先需要先把它加到列表里面去）
2. 筛选出 data、methods、computed 加到 options 上面去。（所以这些数据要遵循相应的写法）
3. 收集 constructor 上的 data
4. 初始化 super 里面的实例属性
5. 处理子类和父类的静态方法
6. 复制使用 reflect 声明的属性

这里需要注意的是，在收集 data 的时候，并不是直接把 data 进行赋值的，因为 data 可以是**一个函数**，所以这里使用**mixins**的方法进行混合。

### vue-property-decorator 的 watch 装饰器的原理

[vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)是基于[vue-class-component](https://github.com/vuejs/vue-class-component)封装的库，它提供了很多方便的装饰器，现在我们来看下它的 watch 装饰器。源码如下：

```js
// vue-class-component 库
export function createDecorator (factory: (options: ComponentOptions<Vue>, key: string, index: number) => void): VueDecorator {
  return (target: Vue | typeof Vue, key?: any, index?: any) => {
    const Ctor = typeof target === 'function'
      ? target as DecoratedClass
      : target.constructor as DecoratedClass
    if (!Ctor.__decorators__) {
      Ctor.__decorators__ = []
    }
    if (typeof index !== 'number') {
      index = undefined
    }
    Ctor.__decorators__.push(options => factory(options, key, index))
  }
}

// vue-property-decorator 库
export function Watch(path: string, options: WatchOptions = {}) {
  const { deep = false, immediate = false } = options

  return createDecorator((componentOptions, handler) => {
    if (typeof componentOptions.watch !== 'object') {
      componentOptions.watch = Object.create(null)
    }

    const watch: any = componentOptions.watch

    if (typeof watch[path] === 'object' && !Array.isArray(watch[path])) {
      watch[path] = [watch[path]]
    } else if (typeof watch[path] === 'undefined') {
      watch[path] = []
    }

    watch[path].push({ handler, deep, immediate })
  })
}
```

这段代码其实就是在 componentOptions 上面**开了一个 watch 属性，用来把各个字符串的 watch 函数推进去**。值得注意的是**执行过程**：

1. 首先执行方法装饰器，把装饰器工厂函数推到```__decorators__```保存起来，此时装饰器**并没有被执行**。
2. 然后执行 component 装饰器，在执行过程中，会把```__decorators__```里面的装饰器**取出，然后执行**，这个时候方法装饰器才生效了。

有一点非常奇怪，因为在 component 装饰器里面，会先把实例方法（就是 watch 的方法）挂载到 methods 里面去，然后再执行方法装饰器，把方法作为 handler 推到相应的 watch 数组里面去。那么这个实例方法不是**没有从 methods 里面删除**吗？看了半天源码也没找到删除的地方，期待大佬解答~~
