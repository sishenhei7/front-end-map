## 概述

很多地方比如官网中需要提升 html 的可访问性，我参考 [element-ui](https://github.com/ElemeFE/element)，总结了一套提升可访问性的方案，记录下来，供以后开发时参考，相信对其他人也有用。

### 可访问性

可访问性基本上分为 2 类：
1. 键盘可访问性：要求页面上的所有可交互的地方均可通过键盘操作。
2. AT可访问性：AT 是 Assistive Technologies 的简写，主要是方便残障用户和网站内容进行交互。

### 键盘可访问性

键盘可访问性有很多内容，但是我觉得主要实现如下 2 点即可：

1.可以使用 tab 来获取焦点。这其中用到的主要方法是 [tabindex](https://developer.mozilla.org/zh-CN/docs/Web/Accessibility/Keyboard-navigable_JavaScript_widgets)。

- tabindex等于 -1：不能使用 tab 键获取焦点，如果元素是隐藏的，那么它的 tabindex 必须设置为 -1；
- tabindex等于 0：能使用 tab 键获取焦点，适用于无顺序的内容
- tabindex等于 xx：能使用 tab 键获取焦点，xx 的值越大，越在前面

示例如下：

```
<!-- 没有tabindex 属性的话, 这些 <span> 元素不会被键盘focus中 -->
<ul>
  <li tabindex="0">
    Include decorative fruit basket
  </li>
  <li tabindex="0">
    Include singing telegram
  </li>
  <li tabindex="0">
    Require payment before delivery
  </li>
</ul>
```

**需要说明的是：**html 里面的 a、button、form 标签即使不添加 tabindex 也能用 tab 键获取焦点。

2.可以使用上、下、左、右键移动选项。比如一个 popover 组件，弹出的内容是可供选择的，那么需要可以使用上、下键进行选择，方法是监听键盘的上、下键，然后监听 enter 键进行选择。示例如下：

```
<template>
  <popper
    trigger="clickToOpen"
    :options="{
      placement: 'top',
      modifiers: { offset: { offset: '0,10px' } }
    }">
    <ul
      class="popper"
      @keydown.down.prevent="navigateOptions('next')"
      @keydown.up.prevent="navigateOptions('prev')"
      @keydown.enter.prevent="selectOption"
    >
      <li tabindex="0">
        Include decorative fruit basket
      </li>
      <li tabindex="0">
        Include singing telegram
      </li>
      <li tabindex="0">
        Require payment before delivery
      </li>
    </ul>

    <button slot="reference">
      Reference Element
    </button>
  </popper>
</template>

<script>
import Popper from 'vue-popperjs';
import 'vue-popperjs/dist/vue-popper.css';

export default {
  components: {
    'popper': Popper
  },
  methods: {
    navigateOptions() {
      // ...
    },
    selectOption() {
      // ...
    },
  },
}
</script>
```

### AT 可访问性

AT 可访问性我觉得也主要分为以下 2 点：

1.尽量使用语义化标签；图像使用 alt 属性描述图像内容；视频使用 title 属性。

2.用 role 来标记这一块 html 是用来干什么的，文档在这里：[文档](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/Main_role)。示例如下：

```
<div id="main" role="main">
  <h1>Avocados</h1>
  <!-- main section content -->
</div>
```

我觉得如下 role 一定需要标出来：

```
navigation、main、article、dialog、search、img、banner、button
```

如果希望更进一步，可以参考 [ARIA文档](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) 使用 ARIA。


