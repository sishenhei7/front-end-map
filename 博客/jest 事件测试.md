## 概述

最近玩 Jest，测试 Vue 组件上的事件，有一些心得，记录下来供以后开发时参考，相信对其他人也有用。

### 事件测试

对于 Vue 组件上的事件，分为 2 种，一种是**子组件 Emit 的事件**，另一种是**插件的事件回调**。

### 子组件 emit 的事件

对于子组件 Emit 的事件，我们使用 Jest mock 这个子组件，然后使用 Vue-Test-Util 提供的方法，**模拟 emit 事件即可**，示例如下：

```
// ChildComponent
export default {
  name: 'ChildComponent',
};

// ParentComponent
<template>
  <div>
    <child-component @custom="onCustom" />
    <p v-if="emitted">Emitted!</p>
  </div>
</template>

<script>
import ChildComponent from './ChildComponent';

export default {
  name: 'ParentComponent',
  components: { ChildComponent },
  data() {
    return {
      emitted: false,
    };
  },
  methods: {
    onCustom() {
      this.emitted = true;
    },
  },
};
</script>

// test.js
import { shallowMount } from '@vue/test-utils';
import ChildComponent from './helper/ChildComponent';
import ParentComponent from './helper/ParentComponent.vue';

describe('emit custom event', () => {
  beforeEach(() => {
    jest.resetAllMocks();
  });

  it("displays 'Emitted!' when custom event is emitted", () => {
    const wrapper = shallowMount(ParentComponent);
    wrapper.find(ChildComponent).vm.$emit('custom');
    expect(wrapper.vm.emitted).toBeTruthy();
    expect(wrapper.html()).toContain('Emitted!');
  });
});
```

### 插件的事件回调

有些插件会提供事件回调，比如 sortable.js 就提供这样的事件回调：

``` js
var sortable = new Sortable(el, {
	group: "name",
	sort: true,
	onSort: function (/**Event*/evt) {
		// 事件回调
	},
});
```

这个时候我们只需要用 Jest mock 这个插件，然后直接**手动执行这个事件绑定的回调函数**即可。因为 Jest 代理了这个插件，所以实际上这个插件并没有真正被引入，所以所有这个插件执行的操作都**需要我们手动调用或者模拟执行**。实例如下：

```
// module.js
function ChildModule(options) {}
export default ChildModule;

// ParentComponent
<template>
  <div>
    <child-component @custom="onCustom" />
    <p v-if="called">Called!</p>
  </div>
</template>

<script>
import ChildModule from './ChildModule';

export default {
  name: 'ParentComponent',
  data() {
    return {
      called: false,
    };
  },
  created() {
    this.childModule = new ChildModule({
      callback: () => {
        this.called = true;
      },
    });
  },
};
</script>

// test.js
import { shallowMount } from '@vue/test-utils';
import ChildModule from './helper/ChildModule';
import ParentComponent from './helper/ParentComponent.vue';

jest.mock('./helper/ChildModule');

describe('emit custom event', () => {
  beforeEach(() => {
    jest.resetAllMocks();
  });

  it("displays 'Called!' when callback is called", async () => {
    const wrapper = shallowMount(ParentComponent);
    ChildModule.mock.calls[0][0].callback();
    expect(wrapper.vm.called).toBeTruthy;
    expect(wrapper.html()).toContain('Called!');
  });
});
```

需要说明的是：我们是**通过这段代码执行回调函数**的：

```
ChildModule.mock.calls[0][0].callback();
```
