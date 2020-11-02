## 概述

最近最近做项目的时候总会思考一些大的**应用设计模式相关**的问题，我把自己的思考记录下来，供以后开发时参考，相信对其他人也有用。

### store里面响应数据变化

通常情况下，我们会把数据存在store里面，并且，有时我们也需要跟踪store里面的数据变化，并作出响应。例子如下：

``` js
export default {
  computed: {
    categories: state => state.categories.categories,
  },
  watch: {
    categories() {
      this.fetchCardData();
    },
  },
  methods: {
    fetchCardData() {
      // 请求卡片数据
    },
  },
}
```

如上所示，当store里面的categories改变的时候，我们会**自动调用api去请求数据**。

### 不响应store里面的数据变化

上面的例子里面，每次当categories改变的时候，fetchCardData方法都会被调用。有些时候，这并不是我们想要的，我们想要的是，当xxxx的时候，categories会改变，fetchCardData方法会跟着被调用；当xxxx的时候，categories会改变，fetchCardData方法又**不会**跟着被调用，怎么办呢？

方法是创造一个标记，但是如何**优雅的**创造标记呢？我有一个方法如下所示：

``` js
// store.js
const state = {
  categories: [],
  categoriesChanges: 0,
};

const actions = {
  updateCategories({ commit }, value) {
    // 如果带有shouldNotChange，则表示不要刷新页面
    if (value.shouldNotChange) {
      commit(types.UPDATE_CATEGORIES, value.data);
    } else {
      commit(types.UPDATE_CATEGORIES, value);
      commit(types.UPDATE_CATEGORIES_CHANGES);
    }
  },
};

const mutations = {
  [types.UPDATE_CATEGORIES](state, value) {
    state.categories = value;
  },
  [types.UPDATE_CATEGORIES_CHANGES](state) {
    state.categoriesChanges += 1;
  },
};

// component.js
export default {
  computed: {
    categories: state => state.categories.categories,
    categoriesChanges: state => state.categories.categoriesChanges,
  },
  watch: {
    categoriesChanges() {
      this.fetchCardData();
    },
  },
  methods: {
    fetchCardData() {
      // 利用this.categories的部分数据来请求卡片数据
    },
  },
}

// business.js
this.$store.dispatch('updateCategories', value); // 会自动调用fetchCardData方法

const payload = {
  shouldNotChange: true,
  data: [...value],
};
this.$store.dispatch('updateCategories', payload); // 不会自动调用fetchCardData方法
```

这样，我们发出同一个action，却能达到**2种不同的效果**，非常方便。


