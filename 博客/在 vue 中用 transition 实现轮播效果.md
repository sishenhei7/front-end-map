## 概述

今天我接到一个需求：**轮播效果**。本来我是打算使用 Swiper 实现的，但是想起来貌似 transition 也能实现。于是就试了下，真的可以，还挺简单的，于是就记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：[进入/离开 & 列表过渡](https://cn.vuejs.org/v2/guide/transitions.html)

### transition

我从官网扒了一个示例的[源码](view-source:https://cn.vuejs.org/v2/guide/transitions.html)，如下所示：

```
<div id="no-mode-translate-demo" class="demo">
  <div class="no-mode-translate-demo-wrapper">
    <transition name="no-mode-translate-fade">
      <button v-if="on" key="on" @click="on = false">
        on
      </button>
      <button v-else="" key="off" @click="on = true">
        off
      </button>
    </transition>
  </div>
</div>
<script>
new Vue({
  el: '#no-mode-translate-demo',
  data: {
    on: false
  }
})
</script>
<style>
.no-mode-translate-demo-wrapper {
  position: relative;
  height: 18px;
}
.no-mode-translate-demo-wrapper button {
  position: absolute;
}
.no-mode-translate-fade-enter-active, .no-mode-translate-fade-leave-active {
  transition: all 1s;
}
.no-mode-translate-fade-enter, .no-mode-translate-fade-leave-active {
  opacity: 0;
}
.no-mode-translate-fade-enter {
  transform: translateX(31px);
}
.no-mode-translate-fade-leave-active {
  transform: translateX(-31px);
}
</style>
```

这个示例是，如果点击按钮，按钮就会从左边渐隐消失，然后另一个按钮会从右边渐隐出现。这不就是轮播效果吗？所以我仿照这个例子做了如下改写：

```
<template>
  <div>
    <div class="chart-wrapper">
      <transition name="slide">
        <div v-if="id === 0" class="chart" key="0">
          <e-charts
            :options="chartOption"
          />
        </div>

        <div v-else-if="id === 1" class="chart" key="1">
          <e-charts
            :options="chartOption"
          />
        </div>
        <div v-else-if="id === 2" class="chart" key="2">
          3333
        </div>
        <div v-else-if="id === 3" class="chart" key="3">
          444
        </div>
      </transition>
    </div>

    <ul style="display: flex;">
      <li @click="id = 0">第一个</li>
      <li @click="id = 1">第二个</li>
      <li @click="id = 2">第三个</li>
      <li @click="id = 3">第四个</li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      id: 0,
    };
  },
  computed: {
    chartOption() {
      return {
        xAxis: {
            type: 'category',
            data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
        },
        yAxis: {
            type: 'value'
        },
        series: [{
            data: [820, 932, 901, 934, 1290, 1330, 1320],
            type: 'line'
        }],
      };
    },
  },
};
</script>

<style lang="scss">
.chart-wrapper {
  position: relative;
  margin-left: 200px;
  width: 800px;
  height: 400px;
}
.chart-wrapper .chart {
  display: flex;
  position: absolute;
  width: 100%;
  height: 400px;
}
.slide-enter-active, .slide-leave-active {
  transition: all 1s;
}
.slide-enter {
  opacity: 0;
  transform: translateX(100%);
}

.slide-leave-active {
  opacity: 0;
  transform: translateX(-100%);
}
</style>
```

上面我们希望通过轮播，来切换 echarts 的图标，但是，实际用起来我们发现，当切换第三页和第四页的时候，切换效果是正常的，说明**已经成功了**。但是在切换第一页和第二页的时候，echarts 图表总是会**无缘无故消失**。

冷静分析，我们在切换的时候，是通过利用 v-if 来实现的，也就是说，v-if 先起作用，然后带动 scss 的动画起作用。那么因为第三页和第四页中的内容是**静态的**，所以 v-if 对它没什么影响；但是第一页和第二页中的 echarts 图表组件，在 v-if 起作用的瞬间，就已经**调用 destroy 方法销毁掉了**，然后 scss 才开始起作用，最后出现轮播的动画效果，所以就出现了 echarts 图表先**消失**，然后才发生轮播动画的情况。

所以这里如果要实现 echarts 图表组件的渐隐，就**不能用 v-if 方法，只能用 v-show 方法**。

### transition-group

如果用 v-show 方法，那么 transition 组件里面就有**不止一个元素**了，所以必须将 transition 改成 transition-group。改后的代码如下：

```
<template>
  <div>
    <div class="chart-wrapper">
      <transition-group name="slide">
        <div v-show="id === 0" class="chart" key="0">
          <e-charts
            :options="chartOption"
          />
        </div>

        <div v-show="id === 1" class="chart" key="1">
          <e-charts
            :options="chartOption"
          />
        </div>
        <div v-if="id === 2" class="chart" key="2">
          3333
        </div>
        <div v-else-if="id === 3" class="chart" key="3">
          444
        </div>
      </transition-group>
    </div>

    <ul style="display: flex;">
      <li @click="id = 0">第一个</li>
      <li @click="id = 1">第二个</li>
      <li @click="id = 2">第三个</li>
      <li @click="id = 3">第四个</li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      id: 0,
    };
  },
  computed: {
    chartOption() {
      return {
        xAxis: {
            type: 'category',
            data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
        },
        yAxis: {
            type: 'value'
        },
        series: [{
            data: [820, 932, 901, 934, 1290, 1330, 1320],
            type: 'line'
        }],
      };
    },
  },
};
</script>

<style lang="scss">
.chart-wrapper {
  position: relative;
  margin-left: 200px;
  width: 800px;
  height: 400px;
}
.chart-wrapper .chart {
  display: flex;
  position: absolute;
  width: 100%;
  height: 400px;
}
.slide-enter-active, .slide-leave-active {
  transition: all 1s;
}
.slide-enter {
  opacity: 0;
  transform: translateX(100%);
}

.slide-leave-active {
  opacity: 0;
  transform: translateX(-100%);
}
</style>
```

由于 transition-group 和 transition 的原理基本上是一样的。所以只需要把 transition 改成 transition-group，然后把 v-if 改成 v-show 就行了，其它地方根本不需要动。

运行起来后，发现 echarts 图表的轮播效果**正常**了！





