## VUE篇

1.MVC 和 MVVM 的不同：MVC 里面的数据模型能够直接渲染到视图上面去，也能通过 Controller 渲染上去，并且在 Controller 中需要手动操作html；而 MVVM 里面的数据只能通过 view-model 与视图进行交互，view-model 作为中间层，核心是双向绑定，不需要人们手动操作 html。

2.Vue 的单向数据流：Vue 的 prop 只能从父级流向子级，而并不能从子级流向父级，这样会防止子级意外改变父级组件的状态，从而导致数据的变化难以理解。

3.双向绑定是指通过修改数据，可以自动实现视图的修改；通过操作视图，又可以反过来自动实现数据的修改。

4.Vue 如何实现双向绑定：Vue 是通过数据劫持和事件来实现双向绑定的，通过劫持数据的 getter 来收集订阅者，通过劫持数据的 setter 来对订阅者进行更新，这样就实现了在数据变动的时候改变视图；然后在操作视图的时候，触发事件，通过事件的回调来改变数据。

5.Vue 如何劫持数组：1.对要监测的数组，拦截它的常用方法，比如splice、push、pop等。2.对于数组单个元素的修改提供$set这个 api。3.遍历数组的元素，如果是对象或者是数组的话，就继续监听。

6.Vue 各生命周期做的事：

```
1.初始化生命周期：初始化父子节点，和各种生命周期flag
2.初始化事件：初始化父节点挂载到它上面的各种事件
3.初始化render相关：挂载createElement，把slot的vnode变成slot对象，添加响应式的 $attrs 和 $listeners
4.=================执行 beforeCreate 钩子========================
5.添加响应式的 injections
6.初始化状态：初始化props、data、compute、watch、methods
7.初始化 provide 属性
8.=================执行 created 钩子=================
9.判断el是否存在，是则调用暴露出来的 $mount 方法（runtime、服务端渲染、非runtime、weex有各自的实现），否则表示是一个组件，等组件自己调用 $mount 方法
10.判断 template 是否存在（不存在的话就去拿 el 的 html），然后编译 template（先编译成 ast 树，然后对静态节点进行优化，最后生成 render 函数代码）
11.=================执行 beforeMount 钩子=================
12.使用 render 函数创建 vnode
13.使用 update 函数把 vnode 生成 $el，并且进行挂载
14.对 vm 建立一个 watcher，当变动的时候，使用 update 方法进行更新
15.=================执行 mounted 钩子=================
16.如果有数据变动
17.=================执行 beforeUpdate 钩子=================
18.使用 update 方法进行更新，并进行 patch
19.=================执行 updated 钩子=================
20.如果开始卸载
21.=================执行 beforeDestroy 钩子=================
22.卸载子组件
23.卸载 watchers、event listeners
24.=================执行 destroyed 钩子=================
```

7.diff算法的理解：

```
比较相同节点下的子节点：
1.先查看有没有相同并且不需要移动的节点，这些节点不需要变动
2.然后查看有没有相同但是需要移动的节点，这部分节点进行简单的移动即可
3.最后对那些没有的旧节点进行删除，对那些没有的新节点进行新增
```

8.vue的组件化原理：我们编写的组件其实就是一个options对象，vue在编译这个组件的时候，会利用vue.extend全局方法生成一个子类，并且合并options，在组件实例化的时候，其实就是这个子类的实例化。所以如果data不是一个函数的话，实例化之后的各个实例就会共享这些data

9.vue、react、angular的优缺点：

```
vue: 基于 html 的模板语法更符合人们的编写习惯，数据响应和组件化也非常容易上手，非常适合中小型项目。缺点是核心库由官方进行维护，发展比较缓慢，数据复杂起来也不好管理，不太适合大型项目；不太兼容低端浏览器。
react: 生态非常丰富，大中小性项目都很适合；缺点是学习难度比较陡峭，对低端浏览器的兼容性比较差。
angular: 有一整套的开发流和规范，比较适合大型项目，对低端浏览器的兼容性也很好。缺点是社区发展没有另外2个好，有一定学习难度，不太适合中小型项目。
```

10.数据频繁变化的时候为什么只会更新一次：因为响应式数据在变化的时候，会在 set 里面把所有依赖它的 watcher 全部更新一遍，但是每个 watcher 在更新的时候实际上没有立即更新，而是会放入一个队列里面去，此时会先根据watcher的id检查这个watcher是不是已经在队列中，如果已经在则不继续放进去，最后在下次 nexttick 的时候才清空队列执行更新。所以当数据频繁变化的时候，所关联的 watcher 都只会有一个在更新队列里面，并在下次 nexttick 的时候更新一次。

11.vue 的 keys 在什么时候会产生错误：vue 的 diff 算法判断 2 个节点相等的时候，只会检查 key、tag、data、是否是注释节点、是否是相同input，而不会检查子节点是否一样。所以如果检查的那几个没有变，而子节点变化了的话，diff算法更新的时候就会发生错误，比较典型的场景是，一串相同节点都有不同的子节点，那么如果删除第一个，则diff算法其实会删除最后一个节点。

12.vue ssr 的原理：在服务端生成 html 字符串发送到客户端，客户端收到后进行客户端激活，把静态 html 转化为动态 html，最后进行挂载。但是这样有一个问题，就是在服务端生成 html 的过程中，vuex 的 store 里面会有数据，这个时候服务端就把里面的数据序列化到```window.__INITIAL_STATE__```里面进行注入让客户端接收。

13.vue diff 算法原理：其实算法的核心原理是对新旧节点进行比对，来尽可能复用 DOM 元素。vue diff 的算法是使用双端比较，从新旧节点的收尾分别建立一个指针向中间靠拢，靠拢的时候首先比较四个指针的节点是否相等，如果有相等则移动即可，没有的话就在老节点的hash表里面找，找到了就继续移动，并把以前的位置赋值为undefined，没找到就创建新的。直到循环结束，然后判断新老节点有没有剩余，新节点有剩余就增加对应节点，老节点有剩余就删除对应节点。

14.前端错误上报：

```
捕获全局错误和资源加载错误：使用 window.addEventListener('error')
捕获vue错误：使用 vue.config.errorHandler
捕获未处理的promise错误：使用window.addEventListener('unhandledRejection')
fetch 和 xhr错误：改写 fetch 方法和 xhr 方法
上报内容：使用 performance api 获取
上报方法：使用 navigator.sendBeacon 方法

改写 fetch 方法：增加then和catch即可
改写 xhr 方法：主要是改写 xhr 的 send 方法，在里面增加监听 error、load 和 abort 的事件
```

15.nexttick 的作用和原理：

```
作用：把回调放到异步队列里面去更新。最常用的场景是，把 dom 更新后才进行的操作放到 nexttick 里面去。
原理：promise -> mutationObserver -> setImmediate -> setTimeout
```

16.performance api 主要有 2 个用途，一个是做性能监控，另一个是检查代码运行时间：

```
性能监控：performance可以得出很多信息：文档load、ready信息，content load信息，tcp连接时间信息，dns查找信息等等

代码运行时间：使用performance.mark和performance.measure api
```

17.检测代码运行时间：

```js
performance.mark('setTimeout-start');

setTimeout(() => {
    performance.mark('setTimeout-end');
    performance.measure('measure', 'setTimeout-start', 'setTimeout-end');
})
```

18.vue 的响应式数据变更到组件渲染的流程：

```
1.响应数据的改变会让setter方法里面的watcher进行更新，其中有一个watcher就是当前组件vm的watcher。
2.vm 的 watcher 在更新的时候会把自己放到一个更新队列里面去，在下一个nextTick的时候更新。
3.vm 的 watcher 在更新的时候会先用 render 方法生成 vnode，然后用 update 方法进行更新，其实就是执行 patch。
4.在 patch 的时候，先判断有没有新旧vnode，只有新vnode那就表示是第一次更新，直接生成元素即可；如果只有旧vnode那就表示是要删除节点。然后判断新旧vnode是否一样，如果不一样就直接用新vnode生成新元素；如果一样则进行patchVnode。
5.patchVnode会先判断是不是相同的vnode，相同则直接返回；然后判断是否是静态节点或文字节点，进行对应的修改；最后对新老节点的children进行diff算法更新。（使用updateChildren进行递归更新）
```

19.客户端激活的过程：

```
1.首先会使用 hydrate 对 html 和 vnode 进行比较判断能否进行激活。
2.能进行激活的话，就使用新 vnode 进行激活，并返回服务端传来的 html
3.不能进行激活的话，就生成一个空的vnode并渲染上去。
```

20.什么是 hippy：是一个跨平台的前端框架，原生支持 react 和 vue。

21.什么是jsbridge: 是 js 和 native 之间的桥梁，主要用于做混合开发。

22.vue 和 小程序有哪些区别：

```
1.生命周期：onlaunch、onShow、onHide、onLoad、onShow、onReady、onHide、onUnload
2.小程序使用 setState
```

23.compute 计算属性的原理：分为三步，首先给这个计算属性初始化一个 watcher，这样的话，这个计算属性会被添加到它依赖的响应式数据中，当响应式数据改变的时候，这个计算属性会更新；然后使用object.defineProperty把这个计算属性代理到vm上面去，并劫持它的getter方法，把这个计算属性的watcher推入响应式数据的依赖中，这样就实现了计算属性的互相依赖；最后在这个getter方法中添加一个dirty属性来标明需不需要重新求值，这就实现了计算属性的缓存。
