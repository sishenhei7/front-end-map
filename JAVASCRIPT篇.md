## Javascript篇

### 概念

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

11.高阶函数：一个函数可以接受另一个函数作为参数或者返回值为一个函数的函数。

12.手写柯里化：

``` js
function currying(func, ...args1) {
    return function(...args2) {
        return func.call(null, ...args1, ...args2);
    };
}
```

13.柯里化的优点：

```
1.参数复用
2.提高适用性
3.延迟执行
```

14.组合继承：

```
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
Function Child(name, age) {
    Parent.call(this, name);
    this.age = age;
}
Child.prototype = new Parent();
```

15.原型链：构造函数有一个原型对象prototype，它的实例有一个属性__proto__就指向这个原型对象，而我们在组合继承当中，这个原型对象prototype又是另一个构造函数的实例，所以它也有一个__proto__属性，指向更上层的原型对象，就这样一直指向Object为止。

16.构造函数的prototype都有一个constructor属性，它指向这个构造函数，所以可以通过这个来判断类与实例的关系，但是constructor属性是可以被重写的，所以我们一般用instanceof来判断类与实例的关系。

17.addEventListener的第三个参数 useCapture 的默认值是false，意思是默认在事件冒泡阶段捕获。（DOM事件流：捕获阶段、目标阶段、冒泡阶段）。可以用 preventDefault 阻止默认行为；使用 stopPropagation 阻止冒泡。

18.浏览器有一个往返缓存 bfcache，用来完全储存前进和后退的网页（此时load事件并不会触发），来加快加载速度。那么怎么在页面上判断是前进或者后退来的呢？可以使用 html5 的 pageshow 和 pagehide 事件。（另外event.persisted可以判断是否来自bfcache）

19.javascript 三大家族：client家族、scroll家族、offset家族。（getBoundingClientRect方法可以获取这些属性）

20.
