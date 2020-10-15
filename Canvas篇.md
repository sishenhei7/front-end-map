## Canvas 篇

1.基本的 canvas

一个基本的 canvas 由 canvas 标签和相应的 js 组成，其中 canvas 标签里面可以加入别的标签，表示如果不支持 canvas 标签的话就显示里面的内容；而 js 部分则通过 getContext 判断支不支持 canvas，从而实现各种操作的。

```js
//html
<canvas id="canvas" width="150" height="150">
  你的浏览器似乎不支持或者禁用了HTML5 <code>&lt;canvas&gt;</code> 元素.
</canvas>

// js
var canvas = document.getElementById('canvas');
if (canvas.getContext){
  var ctx = canvas.getContext('2d');
  // drawing code here
} else {
  // canvas-unsupported code here
}
```

注意，上面是使用 canvas 画 2d 图像的，如果要画 3d 图像的话，可以使用 webGL（也是需要结合 canvas），代码如下：

```js
//html
<canvas id="canvas" width="150" height="150">
  你的浏览器似乎不支持或者禁用了HTML5 <code>&lt;canvas&gt;</code> 元素.
</canvas>

// js
var canvas = document.getElementById('canvas');
if (canvas.getContext){
  var ctx = canvas.getContext('webgl');

  // 确认WebGL支持性
  if (!gl) {
    alert("无法初始化WebGL，你的浏览器、操作系统或硬件等可能不支持WebGL。");
    return;
  }

  // drawing code here
} else {
  // canvas-unsupported code here
}
```

2.矩形和路径

canvas 只支持矩形和路径的绘制，而其它所有图形都可以通过矩形和路径搭配出来：

```js
// 绘制矩形
ctx.fillRect(25, 25, 100, 100);  // 绘制填充的
ctx.clearRect(45, 45, 60, 60);   // 清除区域
ctx.strokeRect(50, 50, 50, 50);  // 绘制边框

// 开始绘制路径
ctx.beginPath();      // 开始绘制
ctx.moveTo(75, 50);   // 从哪儿开始

// 各种路径
ctx.lineTo(100, 25);                        // 绘制直线
ctx.arc(75, 75, 50, 0, Math.PI * 2, true)   // 绘制圆弧
ctx.quadraticCurveTo(25, 25, 25, 62.5);     // 绘制贝塞尔曲线

// 填充还是线条
ctx.fill();     // 填充
ctx.stroke();   // 线条
```

需要注意的是：绘制矩形的参数是x、y、width、height；绘制路径的时候需要先 beginPath，然后进行绘制，最后如果不调用 fill 或者 stroke 是绘制不成功的。

3.添加样式

我们可以给填充物或线条添加各种样式，代码如下：

```js
// 颜色
ctx.strokeStyle = 'rgba(255,0,0,0.5)';
ctx.fillStyle = 'rgba(255,0,0,0.5)';

// 线条样式
ctx.lineWidth = 1+i;
ctx.lineCap = 'round';
ctx.lineJoin = 'round';
ctx.setLineDash([4, 2]);

// 画渐变色
const lingrad = ctx.createLinearGradient(0,0,0,150);
lingrad.addColorStop(0, '#00ABEB');

// 画图案样式
const ptrn = ctx.createPattern(img, 'repeat');

// 画文字和文字样式
ctx.font = "20px Times New Roman";
ctx.fillStyle = "Black";
ctx.shadowOffsetX = 2;
ctx.fillText("Sample String", 5, 30);    // 实体文字
ctx.strokeText("Sample String", 5, 30);  // 边框文字
const text = ctx.measureText("foo");     // 获取文字宽度
text.width; // 16;

// 设置填充规则
ctx.fill("evenodd");
```

需要注意的是：需要先设置样式，然后才能画。

4.图像

我们可以获取各种图像：

```js
const source = document.getElementById('source');
ctx.drawImage(source, 33, 71, 104, 124, 21, 20, 87, 104);
```

需要说明的是：我们首先获取各种源（需要是HTMLImageElement、HTMLVideoElement、HTMLCanvasElement、ImageBitmap中的一个），然后使用 drawImage 画出来即可。

5.变形

在了解变形之前需要了解变形和恢复，一般我们在画某个部分之前，需要先把当前的状态保存下来，然后设置各种状态画这个部分，画完之后恢复使状态回到之前。示例如下：

```js
ctx.save();
ctx.fillStyle = 'rgb(' + (51 * i) + ', ' + (255 - 51 * i) + ', 255)';

// 变形
ctx.translate(10 + j * 50, 10 + i * 50);    // 移动
ctx.rotate(Math.PI*2/(5*6));                // 旋转
ctx.scale(10, 3);                           // 缩放
ctx.transform(cos, sin, -sin, cos, 0, 0);   // 变形
ctx.clip();                                 // 裁剪

ctx.fillRect(0, 0, 25, 25);
ctx.restore();
```

注意，我们可以在一个画布上画2个图堆叠到一起，然后使用 globalCompositeOperation 属性设置堆叠策略。

6.动画

一般来说，我们使用如下步骤画出一帧：

```
1.清空 canvas
2.保存 canvas 状态
3.绘制动画图形（animated shapes）
4.恢复 canvas 状态
```

一般情况下，建议使用 requestAnimationFrame 绘制动画，requestAnimationFrame 的原理是告诉浏览器在下次重绘前使用执行的回调函数更新动画，所以为了持续更新动画，我们需要在回调函数里面继续调用 requestAnimationFrame。基本代码如下。

```js
function draw() {
  ctx.clearRect(0,0,300,300);

  ctx.save();
  // 绘制第一个部分
  ctx.restore();

  ctx.save();
  // 绘制第二个部分
  ctx.restore();

  ctx.save();
  // 绘制第三个部分
  ctx.restore();

  window.requestAnimationFrame(draw);
}

draw();
```

注意：我们可以使用各种公式来设置随时间变化的位置，从而来实现高级动画。

7.其它场景应用

canvas有很多其它场景的应用，详见[canvas教程 mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_usage)：

```
1.像素操作
2.点击区域和无障碍访问
3.保存图片等
```




