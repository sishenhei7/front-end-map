## vue2.x的patch方法和客户端混合

之前确实没自己看过 vue2.x 的 _update 这一块，导致今天被面试官问到了，现在回头补一下这方面的知识。

通过本文，您可以了解到：

1. 怎么进行 patch 的？
2. 怎么用 vnode 创建 DOM 元素的？
3. patchVnode 是怎么进行的？
4. 怎么进行客户端激活？
5. vnode 的一个形象化比喻

### 从初始化 watcher 说起

我们知道，在声明了响应式数据之后，我们再**实例化一个 watcher**并调用响应式数据，才能把 watcher 添加到响应式数据的依赖里面获得更新。而 vue 也是这么做的，它对 vm 来实例化一个 watcher，这个 watcher 在声明的时候会立即对回调函数 updateComponent 求值，从而执行 _update 和 _render，从而把这个 watcher 添加到相应的响应式数据里面。源码如下：

```js
// lifecycle.js
callHook(vm, 'beforeMount');
// ...
updateComponent = () => {
    vm._update(vm._render(), hydrating)
};
// ...
new Watcher(vm, updateComponent, noop, {
    before () {
        if (vm._isMounted && !vm._isDestroyed) {
            callHook(vm, 'beforeUpdate')
        }
    }
}, true /* isRenderWatcher */)
```

这里有一个注意的点，就是这里的 _render 函数其实是调用的 render 函数，而解析 template 并进行静态节点优化以及生成render函数是在 $mount 方法里面进行的，也就是说 template 转化为 render 方法是在 beforeMount 生命周期里面进行的。（如果使用了 vue-loader 的话，那么 vue-loader 里面会直接把 template 打包成 render 方法。）

### _render方法干了些什么呢

_render 方法生成了 vnode，代码如下：

```js
// render.js
Vue.prototype._render = function (): VNode {
    vnode = render.call(vm._renderProxy, vm.$createElement)
    return vnode
}
```

### _update方法干了些什么呢

_update 方法其实就是调用```__patch__```方法，代码如下：

```js
// lifecycle.js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    if (!prevVnode) {
        // initial render
        vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
        // updates
        vm.$el = vm.__patch__(prevVnode, vnode)
    }
}
```

这里如果是初次渲染的话，prevNode 是没有的，所以会拿```vm.$el```这个 DOM 元素去和新的 vnode 进行 patch；如果不是初次渲染的话，就拿旧的 vnode 和 新的 vnode 进行 patch。

### __patch__方法干了些什么呢

```__patch__```方法的源码和解释如下：

```js
function patch (oldVnode, vnode, hydrating, removeOnly) {
    // 如果没有新的 vnode，但有旧的 vnode，就销毁旧的 vnode
    if (isUndef(vnode)) {
      if (isDef(oldVnode)) invokeDestroyHook(oldVnode)
      return
    }

    let isInitialPatch = false
    const insertedVnodeQueue = []

    if (isUndef(oldVnode)) {
      // 如果旧的 vnode 没有，就表示是第一次渲染，则直接创建
      isInitialPatch = true
      createElm(vnode, insertedVnodeQueue)
    } else {
      const isRealElement = isDef(oldVnode.nodeType)
      if (!isRealElement && sameVnode(oldVnode, vnode)) {
        // 如果旧的 vnode 不是 DOM node的形式，并且新旧节点一样，则使用 patchVnode 进行更新
        patchVnode(oldVnode, vnode, insertedVnodeQueue, null, null, removeOnly)
      } else {
        if (isRealElement) {
          // 如果旧的 vnode 是元素节点，并且有服务端渲染标签的话，就移除标签并且进行服务端渲染
          if (oldVnode.nodeType === 1 && oldVnode.hasAttribute(SSR_ATTR)) {
            oldVnode.removeAttribute(SSR_ATTR)
            hydrating = true
          }
          if (isTrue(hydrating)) {
            // 先判断能不能进行混合，能的话，调用 invokeInsertHook 方法进行混合
            if (hydrate(oldVnode, vnode, insertedVnodeQueue)) {
              invokeInsertHook(vnode, insertedVnodeQueue, true)
              return oldVnode
            } else if (process.env.NODE_ENV !== 'production') {
              warn(
                'The client-side rendered virtual DOM tree is not matching ' +
                'server-rendered content. This is likely caused by incorrect ' +
                'HTML markup, for example nesting block-level elements inside ' +
                '<p>, or missing <tbody>. Bailing hydration and performing ' +
                'full client-side render.'
              )
            }
          }
          // 如果混合失败，则使用空的节点
          oldVnode = emptyNodeAt(oldVnode)
        }

        // 如果新旧节点不相同，则使用新的 vnode 进行渲染
        const oldElm = oldVnode.elm
        const parentElm = nodeOps.parentNode(oldElm)

        createElm(
          vnode,
          insertedVnodeQueue,
          oldElm._leaveCb ? null : parentElm,
          nodeOps.nextSibling(oldElm)
        )

        // update parent placeholder node element, recursively
        if (isDef(vnode.parent)) {
          let ancestor = vnode.parent
          const patchable = isPatchable(vnode)
          while (ancestor) {
            for (let i = 0; i < cbs.destroy.length; ++i) {
              cbs.destroy[i](ancestor)
            }
            ancestor.elm = vnode.elm
            if (patchable) {
              for (let i = 0; i < cbs.create.length; ++i) {
                cbs.create[i](emptyNode, ancestor)
              }
              const insert = ancestor.data.hook.insert
              if (insert.merged) {
                for (let i = 1; i < insert.fns.length; i++) {
                  insert.fns[i]()
                }
              }
            } else {
              registerRef(ancestor)
            }
            ancestor = ancestor.parent
          }
        }

        // 移除老的 vnode
        if (isDef(parentElm)) {
          removeVnodes([oldVnode], 0, 0)
        } else if (isDef(oldVnode.tag)) {
          invokeDestroyHook(oldVnode)
        }
      }
    }

    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }
```

这里简单来说分为下面几种情况：

1. 如果没有新的 vnode 只有旧的 vnode，就直接摧毁掉旧的 vnode；
2. 如果没有旧的 vnode，就直接用新的 vnode **创建 DOM 元素**；
3. 对比新旧 vnode，如果是 sameVnode，就开始进行 **patchVnode**；
4. 如果旧的 vnode 是 DOM，并且有 ssr 标记，则清除 ssr 标记并**进行客户端激活**。（如果激活失败则创建一个空的节点）；
5. 如果旧的 vnode 是 DOM，并且没有 ssr 标记，就用新的 vnode 创建 DOM 元素，并且挂载到新的 vnode 的父节点上面。

上面过程中主要有三点需要注意：

1. 怎么用 vnode 创建 DOM 元素的？
2. patchVnode 是怎么进行的？
3. 怎么进行客户端激活？

### 怎么创建 DOM 元素

我们首先来看怎么创建 DOM 元素，下面是 createElm 的源码：

```js
function createElm (
  vnode,
  insertedVnodeQueue,
  parentElm,
  refElm,
  nested,
  ownerArray,
  index
) {
  // 如果这个 vnode 之前用到过，则从缓存里面克隆（注意，这里不是直接用以前的 vnode，因为所有的 vnode 都不能相同）
  if (isDef(vnode.elm) && isDef(ownerArray)) {
    vnode = ownerArray[index] = cloneVNode(vnode)
  }

  vnode.isRootInsert = !nested // for transition enter check
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    // 如果是组件vnode（占位vnode），则使用 createComponent 创建这个组件
    return
  }

  // 如果不是组件 vnode，那证明是 div、span 这种 vnode，那么直接用相应平台的方法进行创建
  const data = vnode.data
  const children = vnode.children
  const tag = vnode.tag
  if (isDef(tag)) {
    if (process.env.NODE_ENV !== 'production') {
      if (data && data.pre) {
        creatingElmInVPre++
      }
      if (isUnknownElement(vnode, creatingElmInVPre)) {
        warn(
          'Unknown custom element: <' + tag + '> - did you ' +
          'register the component correctly? For recursive components, ' +
          'make sure to provide the "name" option.',
          vnode.context
        )
      }
    }

    vnode.elm = vnode.ns
      ? nodeOps.createElementNS(vnode.ns, tag)
      : nodeOps.createElement(tag, vnode)
    setScope(vnode)

    /* istanbul ignore if */
    if (__WEEX__) {
      // in Weex, the default insertion order is parent-first.
      // List items can be optimized to use children-first insertion
      // with append="tree".
      const appendAsTree = isDef(data) && isTrue(data.appendAsTree)
      if (!appendAsTree) {
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue)
        }
        insert(parentElm, vnode.elm, refElm)
      }
      createChildren(vnode, children, insertedVnodeQueue)
      if (appendAsTree) {
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue)
        }
        insert(parentElm, vnode.elm, refElm)
      }
    } else {
      createChildren(vnode, children, insertedVnodeQueue)
      if (isDef(data)) {
        invokeCreateHooks(vnode, insertedVnodeQueue)
      }
      insert(parentElm, vnode.elm, refElm)
    }

    if (process.env.NODE_ENV !== 'production' && data && data.pre) {
      creatingElmInVPre--
    }
  } else if (isTrue(vnode.isComment)) {
    vnode.elm = nodeOps.createComment(vnode.text)
    insert(parentElm, vnode.elm, refElm)
  } else {
    vnode.elm = nodeOps.createTextNode(vnode.text)
    insert(parentElm, vnode.elm, refElm)
  }
}
```

这里简单来说分为下面三种情况：

1. 如果这个 vnode 之前用到过，则从**缓存里面克隆**（注意，这里不是直接用以前的 vnode，因为所有的 vnode 都不能相同）
2. 如果是组件 vnode（占位vnode），则使用 **createComponent** 创建这个组件
3. 如果不是组件 vnode（div、span 这种 vnode），那么**直接用相应平台的方法进行创建**

需要说明的是（源码就不贴出来了）：

1. 为了区分不同的平台（web或者weex），统一使用 modules 和 nodeOps 进行封装平台方法提供统一的接口。其中 modules 主要负责操作 dom 上的属性（class、style、events等），nodeOps 主要负责 DOM 的各种操作（创建、插入、寻找节点等）。
2. insert 方法就是把创建的 DOM 元素插入父节点里面去。

### createComponent

我们来看一下对于组件 vnode，createComponent 方法是怎么进行创建的：

```js
function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
  let i = vnode.data
  if (isDef(i)) {
    const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
    if (isDef(i = i.hook) && isDef(i = i.init)) {
      // 执行组件 vnode 的 init 钩子，
      // 这个钩子会生成一个组件并赋值给 vnode 的 componentInstance 属性，
      // 然后对这个组件执行 $mount 方法进行挂载
      i(vnode, false /* hydrating */)
    }

    // 由于我们在上面生成了 componentInstance 属性，所以我们在这里把组件生成的 DOM 插入到父节点。
    // initComponent 主要处理了 ref 的情况，并对插入时机做了一些调度。
    if (isDef(vnode.componentInstance)) {
      initComponent(vnode, insertedVnodeQueue)
      insert(parentElm, vnode.elm, refElm)
      // 这里是处理 keep-alive 组件的情况
      if (isTrue(isReactivated)) {
        reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
      }
      return true
    }
  }
}
```

这里首先执行组件 vnode 的 init 钩子，这个钩子会生成一个组件并赋值给 vnode 的 componentInstance 属性，然后对这个组件执行```$mount```方法进行挂载。挂载完成后就插入父节点，此时组件 vnode 就创建了 DOM 元素。

需要说明的是：

1. 由于在这里会执行组件的```$mount```方法，所以这里会执行组件的 beforeCreate、created、beforeMount、mounted 生命周期。**这就是为什么子组件的这些生命周期会出现在父组件 mounted 生命周期之前的原因**。
2. 这里组件在执行```$mount```方法的时候又会对自己进行 patch，从而一直 patch 到叶子节点。

### patchVnode 是怎么进行的

我们再来看看相同的 vnode 是怎么 patchVnode 的，下面是 patchVnode 的源码：

```js
function patchVnode (
  oldVnode,
  vnode,
  insertedVnodeQueue,
  ownerArray,
  index,
  removeOnly
) {
  // 新老 vnode 完全一样，则直接返回
  if (oldVnode === vnode) {
    return
  }

  if (isDef(vnode.elm) && isDef(ownerArray)) {
    // 缓存 vnode
    vnode = ownerArray[index] = cloneVNode(vnode)
  }

  const elm = vnode.elm = oldVnode.elm

  if (isTrue(oldVnode.isAsyncPlaceholder)) {
    if (isDef(vnode.asyncFactory.resolved)) {
      hydrate(oldVnode.elm, vnode, insertedVnodeQueue)
    } else {
      vnode.isAsyncPlaceholder = true
    }
    return
  }

  // reuse element for static trees.
  // note we only do this if the vnode is cloned -
  // if the new node is not cloned it means the render functions have been
  // reset by the hot-reload-api and we need to do a proper re-render.
  if (isTrue(vnode.isStatic) &&
    isTrue(oldVnode.isStatic) &&
    vnode.key === oldVnode.key &&
    (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
  ) {
    vnode.componentInstance = oldVnode.componentInstance
    return
  }

  let i
  const data = vnode.data
  // 执行 prepatch 钩子
  if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
    i(oldVnode, vnode)
  }

  const oldCh = oldVnode.children
  const ch = vnode.children
  // 执行 update 钩子
  if (isDef(data) && isPatchable(vnode)) {
    for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
    if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
  }
  // 更新文本
  if (isUndef(vnode.text)) {
    if (isDef(oldCh) && isDef(ch)) {
      // 使用 diff 更新子节点
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
    } else if (isDef(ch)) {
      if (process.env.NODE_ENV !== 'production') {
        checkDuplicateKeys(ch)
      }
      if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
      // 旧的 vnode 的子节点不存在，直接增加新的 vnode 的子节点
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
    } else if (isDef(oldCh)) {
      // 新的 vnode 的子节点不存在，直接删除旧的 vnode 的子节点
      removeVnodes(oldCh, 0, oldCh.length - 1)
    } else if (isDef(oldVnode.text)) {
      nodeOps.setTextContent(elm, '')
    }
  } else if (oldVnode.text !== vnode.text) {
    nodeOps.setTextContent(elm, vnode.text)
  }
  // 执行 postpatch 钩子
  if (isDef(data)) {
    if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
  }
}
```

简单来说，patchVnode 首先会执行 prepatch 和 update 钩子（**用来更新 props、listeners 等**），然后**更新两个节点的文本**，然后对各自的**子节点进行更新**，如果新旧节点的子节点都存在，则使用**diff 算法**进行更新，最后执行 postpatch 钩子。

这里需要注意的是：在对子节点进行 patchVnode 的时候，又会对更深的子节点进行 patchVnode 或创建 DOM 元素，进行循环更新，直到叶子节点。

### 怎么进行客户端激活的

vue 首先会用 hydrate 判断可不可以进行客户端激活，源码如下：

```js
// Note: this is a browser-only function so we can assume elms are DOM nodes.
function hydrate (elm, vnode, insertedVnodeQueue, inVPre) {
  let i
  const { tag, data, children } = vnode
  inVPre = inVPre || (data && data.pre)
  vnode.elm = elm
  // elm 是服务器发回来的DOM，vnode 是客户端创建的 vnode
  // 如果 vnode 是注释节点或静态节点，则直接返回 true
  if (isTrue(vnode.isComment) && isDef(vnode.asyncFactory)) {
    vnode.isAsyncPlaceholder = true
    return true
  }
  // 比较节点的 tag 是否一样
  if (process.env.NODE_ENV !== 'production') {
    if (!assertNodeMatch(elm, vnode, inVPre)) {
      return false
    }
  }
  // 如果 vnode 是组件节点，则使用 init 钩子生成组件节点
  // 在生成组件节点的时候，组件节点自己会调用 patch 方法和 hydrate 方法判断能否激活
  // 所以，只要组件节点生成成功，那么就不需要往下判断了
  if (isDef(data)) {
    if (isDef(i = data.hook) && isDef(i = i.init)) i(vnode, true /* hydrating */)
    if (isDef(i = vnode.componentInstance)) {
      // child component. it should have hydrated its own tree.
      initComponent(vnode, insertedVnodeQueue)
      return true
    }
  }
  // 如果 vnode 是非组件节点，那么递归比较子节点
  if (isDef(tag)) {
    if (isDef(children)) {
      // elm 没有子节点但是 vnode 有子节点，则直接生成子节点插入到 elm 里面
      if (!elm.hasChildNodes()) {
        createChildren(vnode, children, insertedVnodeQueue)
      } else {
        // 比较 vnode 的 v-html 内容是否和 elm 相同
        if (isDef(i = data) && isDef(i = i.domProps) && isDef(i = i.innerHTML)) {
          if (i !== elm.innerHTML) {
            /* istanbul ignore if */
            if (process.env.NODE_ENV !== 'production' &&
              typeof console !== 'undefined' &&
              !hydrationBailed
            ) {
              hydrationBailed = true
              console.warn('Parent: ', elm)
              console.warn('server innerHTML: ', i)
              console.warn('client innerHTML: ', elm.innerHTML)
            }
            return false
          }
        } else {
          // iterate and compare children lists
          let childrenMatch = true
          let childNode = elm.firstChild
          for (let i = 0; i < children.length; i++) {
            if (!childNode || !hydrate(childNode, children[i], insertedVnodeQueue, inVPre)) {
              childrenMatch = false
              break
            }
            childNode = childNode.nextSibling
          }
          // if childNode is not null, it means the actual childNodes list is
          // longer than the virtual children list.
          if (!childrenMatch || childNode) {
            /* istanbul ignore if */
            if (process.env.NODE_ENV !== 'production' &&
              typeof console !== 'undefined' &&
              !hydrationBailed
            ) {
              hydrationBailed = true
              console.warn('Parent: ', elm)
              console.warn('Mismatching childNodes vs. VNodes: ', elm.childNodes, children)
            }
            return false
          }
        }
      }
    }
    // 比较 elm 和 vnode 的 data
    if (isDef(data)) {
      let fullInvoke = false
      for (const key in data) {
        if (!isRenderedModule(key)) {
          fullInvoke = true
          invokeCreateHooks(vnode, insertedVnodeQueue)
          break
        }
      }
      if (!fullInvoke && data['class']) {
        // ensure collecting deps for deep class bindings for future updates
        traverse(data['class'])
      }
    }
  } else if (elm.data !== vnode.text) {
    // 比较 elm 和 vnode 的文本（如果不一样则直接使用vnode的文本）
    elm.data = vnode.text
  }
  return true
}
```

这里主要做了下面几件事：

1. 把 elm 赋值给 ```vnode.elm``` 储存起来。
2. 比较 elm 和 vnode 的 tag 是否一样。（生产环境会跳过）
3. 如果 vnode 是组件节点，则使用 init 钩子生成组件节点。（在生成组件节点的时候，组件节点自己会调用 patch 方法和 hydrate 方法判断能否激活，所以，只要组件节点生成成功，那么就不需要往下判断了）
4. 如果 vnode 是非组件节点，那么递归比较子节点。
5. 比较 elm 和 vnode 的 data
6. 比较 elm 和 vnode 的文本（如果不一样则直接使用vnode的文本）

需要说明的是：

1. **生产环境**会忽略掉大部分错误，直接使用 vnode 对 elm 进行矫正。
2. 为什么要进行客户端激活？因为如果**直接把 vnode 生成 DOM 并挂载是非常耗时间和内存的**，所以这里激活的过程就相当于给 elm 使用 vnode 进行 patch 的过程。
3. 执行 hydrate 方法之后，其实就已经激活完成了，后面会直接使用```invokeInsertHook(vnode, insertedVnodeQueue, true)```进行插入了。
4. ```invokeInsertHook```方法会把```createElm```里面的```insert```方法缓存起来，最后在整个 root vnode 更新结束后，再一起执行所有的```insert```方法。

### 通俗的认识 vnode

vnode 其实并不神秘，它其实就是**一堆数据**和**它的皮（elm）**组成，它的皮（elm）是指挂载的 DOM，所以每次 patch 的时候会深度遍历比较它的数据，然后更新它的皮（elm）；而在客户端激活的时候，只是 vnode 为了尽可能**复用服务器发回来的 html**（因为DOM操作很昂贵），而在服务器发回来的 html 上修修补补形成一个新的皮而已！！！





