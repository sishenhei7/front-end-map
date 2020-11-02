## 概述

今天 PM 说，需要把 echarts 图表的纵坐标调成这样：**如果全是 4 位数就用 K 为单位**。冷静分析，就是说如果纵坐标刻度的间距是四位数，就用 K 为单位。那要如何获取纵坐标刻度的间距呢？我们都知道，**Echarts 纵坐标刻度的间距是它自己生成的，而且会变**。于是我去**看 Echarts 源码**碰碰运气，竟然让我发现了 Echarts 的纵坐标刻度的间距的生成方法！！！记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[Echarts源码1](https://github.com/apache/incubator-echarts/blob/fd064123626c97b36cbd6da1b5fc73385c280abd/src/util/number.js)

[Echarts源码2](https://github.com/apache/incubator-echarts/blob/fd064123626c97b36cbd6da1b5fc73385c280abd/src/scale/Interval.js)

### 源码

简单来说，它是用如下方法生成的：

``` js
// this.data 是数据
const round = true;
const splitNumber = 4;
const max = this.data.reduce((x,y) => x > y ? x : y);
let val = max / splitNumber;

// echart 内部计算 interval 的方法
// https://github.com/apache/incubator-echarts/blob/fd064123626c97b36cbd6da1b5fc73385c280abd/src/util/number.js
const exponent = Math.floor(Math.log(val) / Math.LN10);
const exp10 = Math.pow(10, exponent);
const f = val / exp10; // 1 <= f < 10
let nf;
if (round) {
  if (f < 1.5) { nf = 1; }
  else if (f < 2.5) { nf = 2; }
  else if (f < 4) { nf = 3; }
  else if (f < 7) { nf = 5; }
  else { nf = 10; }
}
else {
  if (f < 1) { nf = 1; }
  else if (f < 2) { nf = 2; }
  else if (f < 3) { nf = 3; }
  else if (f < 5) { nf = 5; }
  else { nf = 10; }
}
val = nf * exp10;

// Fix 3 * 0.1 === 0.30000000000000004 issue (see IEEE 754).
// 20 is the uppper bound of toFixed.
return exponent >= -20 ? +val.toFixed(exponent < 0 ? -exponent : 0) : val;
```

可以看到：

1. 间距只和**数据的最大值**还有 **splitNumber** 有关，与其它值都没有关系。
2. Echarts 使用 **log10** 来获取最优的间距。具体算法原理不知道。
3. Math.log(val) / Math.LN10 是 lgx(**以10为对数**)；Math.log(val) / Math.LN2 是 lgx(**以2为对数**)
