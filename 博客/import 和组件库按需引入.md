## 概述

今天查资料查到了一些**有趣的东西**，记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[import、require、export、module.exports 混合使用详解](https://segmentfault.com/a/1190000012386576)

### 从 import 讲起

```import { button, Select } from 'element-ui'```这段代码到底发生了什么？

babel 会将这段代码进行**转码**，转码之后是这个样子：

```
var a = require('element-ui');
var Button = a.Button;
var Select = a.Select;
```

所以，就算我们只想使用 element-ui 的 button 和 Select 这2个组件，但是实际上，我们把**整个 elment-ui 库**引入进来了。

### babel-plugin-component 做了什么

我们知道，在 element-ui 的文档里面强调，如果要使用按需加载，就要使用 babel-plugin-component 库。那么 babel-plugin-component 库又做了什么？简单来说，它把上面的代码转化成了下面的样子：

```
import Button from 'element-ui/lib/button'
import Select from 'element-ui/lib/select'
```

这样就只会引入 Button 和 Select 这两个组件了。

类似的，e-charts 在按需引入组件的时候是这么引入的，也是一样的道理。

```
import 'echarts/lib/chart/bar';
import 'echarts/lib/chart/line';
import 'echarts/lib/chart/scatter';
import 'echarts/lib/chart/effectScatter';
import 'echarts/lib/chart/treemap';
```

所以，所有**支持按需引入的库**，都可以用**上面的方法**进行按需引入组件。

### 一个问题

如果我们有一个需求，就是在 app 加载的时候不引入 element-ui 库，然后在```/admin```这个路由下面才引入 element-ui ,那要怎么做呢？

在 main.js 使用各种形式的按需加载肯定是不行的，因为会打包进入 **app.js** 里面去，从而在初次加载 app 的时候加载。

在```/admin```这个路由下，各个组件用到 element-ui 的组件的时候，各自按需加载。这个方法是可行的，但是非常繁琐，每次用到 element-ui 的组件都需要引入一下，然后挂载一下。

我选择的方法是，在```/admin```这个路由的组件下面，按需加载全部需要的 element-ui 组件，示例代码如下：

```
// admin.element.js
import Vue from 'vue';
import Row from 'element-ui/lib/row';
import Col from 'element-ui/lib/col';
import Table from 'element-ui/lib/table';
import TableColumn from 'element-ui/lib/table-column';
import Button from 'element-ui/lib/button';
import Footer from 'element-ui/lib/footer';
import Form from 'element-ui/lib/form';
import FormItem from 'element-ui/lib/form-item';
import Input from 'element-ui/lib/input';
import Menu from 'element-ui/lib/menu';
import Submenu from 'element-ui/lib/submenu';
import MenuItem from 'element-ui/lib/menu-item';
import Loading from 'element-ui/lib/loading';
import Message from 'element-ui/lib/message';

const components = [
  Row,
  Col,
  Table,
  TableColumn,
  Button,
  Footer,
  Form,
  FormItem,
  Input,
  Menu,
  Submenu,
  MenuItem,
];
components.forEach((component) => {
  Vue.component(component.name, component);
});

Vue.use(Loading.directive);
Vue.prototype.$message = Message;
```

### 内存泄漏

粗看上面的代码是没有问题的，但是仔细想的话，如果每次跳转到```/admin```路由下面的话，那不是每次都会安装引入的这些组件吗？那**如果跳转多次就会安装多次**，造成**内存泄漏**啊~~~

虽然说 Vue 本身的插件安装机制已经避免了这种情况，但是我们还是建议加上一行**判断代码**：
```
import Vue from 'vue';
import Row from 'element-ui/lib/row';
import Col from 'element-ui/lib/col';
import Table from 'element-ui/lib/table';
import TableColumn from 'element-ui/lib/table-column';
import Button from 'element-ui/lib/button';
import Footer from 'element-ui/lib/footer';
import Form from 'element-ui/lib/form';
import FormItem from 'element-ui/lib/form-item';
import Input from 'element-ui/lib/input';
import Menu from 'element-ui/lib/menu';
import Submenu from 'element-ui/lib/submenu';
import MenuItem from 'element-ui/lib/menu-item';
import Loading from 'element-ui/lib/loading';
import Message from 'element-ui/lib/message';

// hack: 防止多次引入导致内存泄漏
if (!Vue.hasAdminPageImportedElement) {
  const components = [
    Row,
    Col,
    Table,
    TableColumn,
    Button,
    Footer,
    Form,
    FormItem,
    Input,
    Menu,
    Submenu,
    MenuItem,
  ];
  components.forEach((component) => {
    Vue.component(component.name, component);
  });

  Vue.use(Loading.directive);
  Vue.prototype.$message = Message;

  Vue.hasAdminPageImportedElement = true;
}

```


