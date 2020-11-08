# vue2.x源码中的占位符

事情的起因是我再次看了这篇掘金文章[从一次 vue ssr 渲染客户端报错, 来看 ssr 客户端激活过程](https://juejin.im/post/6844904029315678221)，里面写的```在 updateClass() 中, vnode 的 tag 是 div, 而 vnode 的 elm 却是 comment. 因为 comment 节点是没有 setAttribute 方法的, 所以就报错了.```让我看的不是很懂，特别是 comment 是干什么的我不是很懂，所以我搭了一个项目进行调试，理顺我不清楚的地方。

通过本篇文章，您可以学到：

1. 怎么搭环境调试 vue2.x 源码？
2. 占位符是什么？有什么用？
3. 组件更新阶段是怎么更新占位符节点的？

## 环境搭建

1.我们使用 vue-cli **搭建一个项目**：

```bash
vue create test-sourcecode
```

2.在项目里面**克隆 vue 源码库**：

```bash
cd test-sourcecode
git clone git@github.com:vuejs/vue.git
```

3.进入 vue 源码库**并使用 watch 形式编译**：

```bash
cd vue-dev
npm install
npm run dev
```

4.回到 test-sourcecode 文件夹，加入```.eslintignore```文件，**禁止 eslint 检查 vue 源码**：

```bash
cd ..
echo 'vue-dev' >> .eslintignore
```

5.把项目中的```import Vue from 'vue'```替换为```import Vue from '../vue-dev/dist/vue.js'```

6.使用```npm run serve```启动项目即可。在源码和项目上的改动都会有**热重载效果**，可以放心加```console.log```看输出了。

## 占位符是什么？有什么用？

我们经常在各种源码分析文章上面看到**占位符节点、placeholder节点**等，他们是干什么的呢？

这里我就不贴图或者代码了，直接说了。vue 的组件在生成 vnode 的过程中，对于原生节点就直接生成 tag 为对应原生 tag 的**vnode 节点**，而对于组件节点就生成 tag 是```vue-component-1-App```这种类型的节点，这种节点就是占位符节点，或者叫 placeholder 节点。

这种节点的作用是，**只是占据一个位置而已，并不会像原生vnode节点一样会进行更新，也不会带上对应组件vnode的数据**，它的更新流程是这样的，在比较新旧占位符节点进行更新的时候：

1. 如果新旧占位符节点**有不存在**的，就直接创建或销毁占位符节点，同时通过vnode hook创建或销毁对应的组件 vnode。
2. 如果新旧占位符节点**都存在，但是不是同一个节点**，就销毁旧节点，在旧占位符节点的父节点下创建新占位符节点，同时创建对应的组件 vnode。
3. 如果新旧占位符节点**都存在，并且是同一个节点**，则使用vnode hook更新对应的组件 vnode。

注意：上面提到的组件 vnode 指的是，占位符对应的组件生成的组件 vnode，它才是真正控制这个组件更新的 vnode。(另外，函数式组件不会生成占位符节点，因为它没有生命周期)

那怎么**通过占位符节点找到对应的组件 vnode 呢**？其实很简单，在创建占位符节点的时候，会把组件 vnode 挂载到这个占位符节点的**componentInstance**属性上，所以这个属性就指向了组件 vnode。

## 组件更新节点是怎么更新占位符节点的

具体的更新流程上面已经说了，总的来说，就是使用占位符节点的 vnode hook 来更新组件 vnode ，这里详细说明一下**怎么用 vnode hook 来更新的**。

1.判断这个节点是不是占位符节点，源码中是**通过查看占位符节点有没有 hook 进行判断**的，如果是的话，就进行 prepatch:

```js
if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
  i(oldVnode, vnode)
}
```

2.在进行 prepatch 的时候，会**通过 componentInstance 拿到对应的组件 vnode**，然后通过 updateChildComponent 方法**更新组件 vnode 的props、listeners等**。

```js
prepatch (oldVnode: MountedComponentVNode, vnode: MountedComponentVNode) {
    const options = vnode.componentOptions
    const child = vnode.componentInstance = oldVnode.componentInstance
    updateChildComponent(
        child,
        options.propsData, // updated props
        options.listeners, // updated listeners
        vnode, // new parent vnode
        options.children // new children
    )
},
```

3.到这里占位符节点的**任务就已经完成**了，但是组件只更新了props、listeners等，然后组件**自身通过props、listeners这些响应式数据，来实现自身的更新**。

## 其它

在看源码的时候，还是有几个小问题的，留待下次解决了：

1. 什么是**comment 节点**？
2. 什么是**asyncPlaceholder**?
3. 什么是**static vnode**?
