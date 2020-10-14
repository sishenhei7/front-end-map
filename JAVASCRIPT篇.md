## Javascript篇

### 基础

1.概念：狭义的 js 只是 ecmascript，描述了js的语法等；但是由于 js 与浏览器关系密切，所以就有了浏览器特定的 js 扩展：BOM（浏览器对象模型），两者一起就是广义的 js。BOM 里面有一个 DOM（文档对象模型），描述了文档对象的一些东西，除此之外 BOM 还包括 location，navigator，history 等等非文档的东西。

1.原始数据类型：boolean, number, object, array, null, undefined, symbol, bigint

2.对于原始数据来说，typeof 除了 null 都会显示正确的类型（这是 js 的一个 bug，typeof null 会显示 object）；对于引用类型，除了 function 都会显示 object

3.instanceof 常用来判断一个对象在其原型链中是否存在一个构造函数的 prototype 属性。

4.Object.is 修复了 === 的一些问题，比如 -0 === +0 等。

5.js 中的类型转换：转换成数字、转换成布尔值、转换成字符串。=== 和 == 的区别不仅在于 == 不会要求类型相等，还会自动转换类型。

6.对象转原始类型的顺序：Symbol.toPrimitve方法；valueOf方法；toString方法；报错。（所以可以利用这个让if(a == 1 && b == 2)的值为true）

7.浅拷贝是指一个新的对象直接拷贝已存在的对象的对象属性的引用。可以浅拷贝的方法：Object.assign、...展开、concat拷贝数组、slice浅拷贝。

8.json.parse进行深拷贝的局限性：无法解决循环引用；无法拷贝一些特殊的对象，比如函数、Date等。

9.简易手写深拷贝：

``` js
function deepcopy(obj, cache = []) {
    if (typeof obj === null || type obj !== 'object') {
        return obj;
    }

    const hit = cache.find(item => item === obj);

    if (hit) {
        return hit;
    }

    cache.push(obj);

    const copy = Array.isArray(obj) ? [] : {};

    Object.keys(obj).forEach((key) => {
        copy[key] = deepcopy(obj[key]);
    });

    return copy;
}
```

10.在 foreach 中使用 return 是没有效果的，所以中断不了 foreach 的循环；因此官方推荐使用 some 或 every。

11.let、const的特性：1.不存在变量提升；2.不允许重复声明；3.块级作用域

12.Object相关方法：

```
Object.keys: 是对键名的遍历
Object.values: 是对键值的遍历
Object.entries: 是对键值对的遍历
```

13.手写防抖和节流

```js
// throttle
function throttle(fn, interval = 300) {
    let lock = false;

    return function (...args) {
        if (lock) return;

        lock = true;
        setTimeout(() => {
            lock = false;
            fn(...args);
        }, interval);
    }
}

// debounce
function debounce(fn, interval = 300) {
    let timeout = null;

    return function (...args) {
        if (timeout) clearTimeout(timeout);

        timeout = setTimeout(() => {
            fn(...args);
        }, interval);
    }
}
```

14.手写原生 xhr 发送请求：

```js
let Req = new XMLHttpRequest();
Req.onload = function(res) {
    console.log(res);
};
Req.open('get', url);
Req.responseType = 'json';
Req.send();
```

### 函数式

1.高阶函数：一个函数可以接受另一个函数作为参数或者返回值为一个函数的函数。

2.手写柯里化：

``` js
function currying(func, ...args1) {
    return function(...args2) {
        return func.call(null, ...args1, ...args2);
    };
}
```

3.柯里化的优点：

```
1.参数复用
2.提高适用性
3.延迟执行
```

4.手写函数记忆：

```js
let memorize = (func, content) => {
    let cache = Object.create(null);
    content = content || this;
    return (key) => {
        if (!cache[key]) {
            cache[key] = func.call(content, key);
        }

        return cache[key];
    }
}
```

5.手写 promise.all 和 promise.race：

```js
Promise.prototype.all = function (iterator) {
    let num = 0;
    const result = [];
    const len = iterator.length;

    return new Promise((resolve, reject) => {
        for (item in iterator) {
            Promise.resolve(item)
                .then((res) => {
                    num += 1;
                    result.push(res);

                    if (num >= len) {
                        resolve(result);
                    }
                })
                .catch((err) => {
                    reject(result);
                });
        }
    });
}

Promise.prototype.race = function (iterator) {
    return new Promise((resolve, reject) => {
        for (item in iterator) {
            Promise.resolve(item)
                .then((res) => {
                    resolve(res);
                })
                .catch((err) => {
                    reject(err);
                });
        }
    });
}
```

6.手写flaten:

```js
// [1, [2, 3, [[4, 5, [6, 7], 8], 9, 10, 11]]]
// 递归形式
function flatten(arr) {
    const result = [];
    arr.forEach(item => Array.isArray(item) ? result.push(...flaten(item)) : result.push(item));
    return result;
}

// 使用tostring
function flatten_new(arr) {
    return arr.toString().split(',');
}

// 尾递归
function flaten_tail(arr) {
    const helper = (first, rest, result) => {
        if (!Array.isArray(first)) {
            result.push(first);

            if (rest.length === 0) {
                return result;
            }

            return helper(rest, [], result);
        } else {
            const [newFirst, ...newRest] = first.concat(rest);
            return helper(newFirst, newRest, result);
        }
    }

    return helper(arr, [], []);
}
```

### 类与原型链

1.组合继承：

```js
function Parent (name) {
  this.name = name;
  this.colors = ['red', 'blue', 'green'];
}
Parent.prototype.getName = function () { return this.name; }

// 原型链继承
// 能够继承父类的所有原型，但是继承不了属性，同时也会把父类的属性和方法也复制过来到原型链里面
Function Child1(name, age) {
    this.age = age;
}
Child1.prototype = new Parent();

// 构造函数继承
// 可以继承父类的属性，但是继承不了 prototype
Function Child2(name，age) {
    Parent.call(this, name);
    this.age = age;
}

// 组合继承
// 把上面结合起来
// 改进：Child.prototype = Object.create(Parent.prototype);
Function Child(name, age) {
    Parent.call(this, name);
    this.age = age;
}
Child.prototype = new Parent();
```

2.原型链：构造函数有一个原型对象prototype，它的实例有一个属性__proto__就指向这个原型对象，而我们在组合继承当中，这个原型对象prototype又是另一个构造函数的实例，所以它也有一个__proto__属性，指向更上层的原型对象，就这样一直指向Object为止。

3.构造函数的prototype都有一个constructor属性，它指向这个构造函数，所以可以通过这个来判断类与实例的关系，但是constructor属性是可以被重写的，所以我们一般用instanceof来判断类与实例的关系。

### bom相关

1.addEventListener的第三个参数 useCapture 的默认值是false，意思是默认在事件冒泡阶段捕获。（DOM事件流：捕获阶段、目标阶段、冒泡阶段）。可以用 preventDefault 阻止默认行为；使用 stopPropagation 阻止冒泡。

2.浏览器有一个往返缓存 bfcache，用来完全储存前进和后退的网页（此时load事件并不会触发），来加快加载速度。那么怎么在页面上判断是前进或者后退来的呢？可以使用 html5 的 pageshow 和 pagehide 事件。（另外event.persisted可以判断是否来自bfcache）

3.javascript 三大家族：client家族、scroll家族、offset家族。（getBoundingClientRect方法可以获取这些属性）

4.数据结构的栈是指后进先出的数据结构，队列是指先进先出的数据结构，树是指具有根节点并且其它节点都是子树节点的数据结构，堆是指满足父子节点大小关系的完全二叉树。操作系统的栈和堆都是指内存空间，不同是的，堆更大，并且按需申请、动态分配内存。

5.垃圾回收机制：原始数据储存在栈空间中，引用类型数据储存在堆空间中。在函数执行上下文之后，就立即销毁栈中的数据（如果这个数据放在闭包里面，则没有放到栈空间而是放到堆空间中）。堆分为新生代和老生代两个区域，它们使用标记-清除的方式进行回收的。在执行上下文创建的时候进行标记，在执行上下文执行完毕的时候进行回收。

### 闭包

1.概念：闭包就是定义在一个函数内部的函数，它在外部函数被回收之后仍然能够读取外部函数中的变量。

2.闭包的作用：模仿块级作用域（进行代码封装）；私有变量

3.作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。Javascript 采用的是词法作用域，也就是静态作用域。

### 事件循环

1.事件循环：在 js 中，大部分任务是在主线程上执行的，而这些任务有轻重之分，需要对它们的执行顺序做一定的安排，所以v8使用队列的方式储存这些任务，让它们交替执行。

2.过程：事件循环遵循宏任务-清空自己的微任务队列-ui渲染-宏任务...的方式执行的，常见的宏任务有各种事件、setTimeout等。常见的微任务有Promise、MutationObserver等等。每个宏任务都有自己的微任务队列。

3.微任务出现的原因：由于宏任务的时间粒度比较大，执行的时间不能精确控制，对一些实时性比较高的需求就不太符合了。

4.MutationObserver 接口提供了监视对 DOM 树所做更改的能力，它被设计为旧的 Mutation Events 功能（本质是宏任务）的替代品。

5.requestAnimationFrame 是一个动画函数，并且让浏览器在下次重绘之前调用指定的回调函数更新动画。它常常被用来作为 setTimeout 的替代品来创建动画。

6.MutationObserver 和 requestAnimationFrame 出现的原因：因为它们之前是使用宏任务实现的，但是在宏任务之后执行另一个宏任务的时候，中间有一段 ui 渲染的过程，特别是当执行多个宏任务的时候，中间有非常多 ui 渲染的过程，非常不必要。所以使用微任务来实现，此时只会把微任务放到微任务队列里面去，并且在 ui 渲染之前执行。

### 尾调用优化

1.尾调用是指一个函数的最后一步是调用另一个函数。必须是一个函数，不能是一个函数表达式或者函数引用。

2.尾调用优化的原因：外层函数在调用的时候，会在内存中形成一个调用栈，然后再调用内层函数的时候，会再压入内层函数的调用栈，在调用完内层函数的时候，会释放内层函数的调用栈，最后再释放外层函数的调用栈。可以看到，外层函数的调用栈是一直存在于内存中的。但是如果内层函数是一个函数的话，其实可以释放外层函数的调用栈的，所以可以大大减少内存的占用。

3.注意：1.实现尾调用优化的方法是把外层函数的变量放到内层函数的参数里面去，所以可能需要多加几个参数（使用参数的默认值解决）。2.只有在严格模式下才能进行尾调用优化。

4.斐波拉切数的尾调用优化递归版本：

```js
// 求第 n 位斐波拉切数
function fibbo(n, before = 0, temp = 1) {
    if (n <= 1) return temp;
    return fibbo(n - 1, temp, before + temp);
}
```

### 项目中用到的 es6

```
1.let、const这些关键字
2.变量的解构赋值
3.箭头函数
4.数组的foreach、includes、find等方法，对象的for of等方法
5.模板字符串
6.promise
7.class
```
