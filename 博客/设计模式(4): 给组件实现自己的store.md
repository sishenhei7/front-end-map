## 概述

最近最近做项目的时候总会思考一些大的**应用设计模式相关**的问题，我把自己的思考记录下来，供以后开发时参考，相信对其他人也有用。

### 组件自身的store

我们在开发组件的时候，时常都有这种需求，就是希望**给组件一个独立的store**，这个store可能被用来储存数据，共享数据，还可以被用来对数据做一些处理，抽离核心代码等。

### store的数据不共享

如果组件自身的store是**每个实例独自拥有的并且不共享**的话，我们可以直接用一个类来实现。

``` js
// store.js
export default class Store {
  constructor(data, config) {
    this.config = config;
    this.init(data);
  }

  init(data) {
    // 对数据做处理
  }

  // 其它方法
}
```

然后我们在组件中实例化这个store，然后挂载到data属性里面去：

``` js
<script>
import Store from './store';

export default {
  data() {
    return {
      store: [],
    };
  },
  methods: {
    initStore() {
      // 生成 options 和 config
      this.store = new Store(options, config);
    },
  },
};
</script>
```

### store的数据需要共享

如果store的**数据需要共享**，我们建议用动态挂载vuex的store的方法，示例如下：

``` js
// store.js
const state = {
  data: [],
};

const getters = {};

const actions = {};

const mutations = {
  setData(state, value) {
    this.state.data = [...value];
  },
};

export default {
  state,
  getters,
  actions,
  mutations,
};
```

然后我们在注册这个组件的时候动态挂载这个store：

``` js
import Store from './store';

export default {
  install(Vue, options) {
    Vue.store.registerModule('xxx', store);
  },
};
```

最后我们就可以在组件中使用这个store的数据啦~~~
