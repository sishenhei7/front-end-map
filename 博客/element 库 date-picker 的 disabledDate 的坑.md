## 概述

今天使用 element 库的 date-picker 组件，使用日期范围，然后使用了 disabledDate 属性，把 2018 年 1 月和 2020 年 12 月之后的日期全部 disable 掉的时候，出现了一个小坑，就是点击2018 年 1 月和 2020 年 12 月的时候，**点击会变得不顺畅，而且有时候会发生点击无效的情况**，点击其它日期是正常的。我试了多种方法，终于把它解决了，记录下来，供以后开发时参考。

### 有坑的代码

下面是我原先点击不顺畅的代码：

``` vue
<template>
  <div class="block">
    <el-date-picker
      v-model="value"
      type="monthrange"
      align="right"
      range-separator="至"
      start-placeholder="开始月份"
      end-placeholder="结束月份"
      :picker-options="pickerOptions">
    </el-date-picker>
  </div>
</template>
<script>
export default {
  data() {
    return {
      pickerOptions: {
        disabledDate: (time) => {
          return time.getTime() < new Date('2018-01').getTime()
            || time.getTime() > new Date('2020-12').getTime();
        },
      },
      value: '',
    };
  },
};
</script>
```

分析原因，可能是 getTime 的锅，我把 time.getTime() 打印出来，发现组件上的 2018 年 1 月的getTime() 值略微大于 new Date('2018-01').getTime()。所以我想可能**问题在于getTime**，于是我换了一种实现：

``` vue
<template>
  <div class="block">
    <el-date-picker
      v-model="value"
      type="monthrange"
      align="right"
      range-separator="至"
      start-placeholder="开始月份"
      end-placeholder="结束月份"
      :picker-options="pickerOptions">
    </el-date-picker>
  </div>
</template>
<script>
export default {
  data() {
    return {
      pickerOptions: {
        disabledDate: (time) => {
          this.isEarlyThan(time, new Date('2018-01'))
            || this.isEarlyThan(new Date('2020-12'), time);
        },
      },
      value: '',
    };
  },
  methods: {
    isEarlyThan(dateA, dateB) {
      // dateA 比 dateB 早则为 true(严格小于)
      const yearA = dateA.getFullYear();
      const yearB = dateB.getFullYear();

      if (yearA < yearB) {
        return true;
      } if (yearA === yearB) {
        return dateA.getMonth() < dateB.getMonth();
      }

      return false;
    },
  },
};
</script>
```

结果问题成功解决！！！

### 另外

另外我发现，如果不指明小时的话，**getHours() 会返回 8**。示例如下：

```
new Date('2018-03-01').getHours() // 8
new Date('2018-03').getHours() // 8

new Date(2018, 1, 1, 0, 0, 0).getHours() // 0
```

所以以后在使用 getHours() 方法的时候需要格外小心，element 库的坑的原因或许也是因为这个。

### 性能

点击不通畅的问题也可能是性能原因，所以我们比较一下使用 getTime 和使用 isEarlyThan 方法的性能快慢，通过分析代码，可以知道其实主要是比较 getTime 和 getFullYear + getMonth 的性能大小。代码如下：

```
const date1 = new Date('2018-01');
const date2 = new Date();

console.time('getTime');
date1.getTime() > date2.getTime();
console.timeEnd('getTime');

console.time('getFullYear + getMonth');
date1.getFullYear() > date2.getFullYear();
date1.getMonth() > date2.getMonth();
console.timeEnd('getFullYear + getMonth');
```

结果 getTime 的性能比 getFullYear + getMonth 的性能要高一些。所以看来使用 getTime 的卡顿现象是组件自身的原因了。
