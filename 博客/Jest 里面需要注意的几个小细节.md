## 概述

最近学 jest ，有一些细节记录下来，供以后开发时参考，相信对其他人也有用。

### import 提升

ES6 的 import 会自动提升到文档前面，所以下面的 import 会提升到前面。

```
let wrapper;
import Sortable from "sortablejs";
```

注意：如果不希望 import 提升，方法有两种，一种是利用 require 引入；另一种是使用 babel-plugin-dynamic-import-node 插件动态 import 引入。

### jest.mock 提升

因为 jest.mock 是用来 mock 模块的，于是 jest 也会将 jest.mock 进行提升，并且把它提升到 import 的前面。示例代码如下：

```
import Sortable from "sortablejs";
jest.mock('sortablejs');
```

上线的代码相当于：

```
jest.mock('sortablejs');
const sortable = require('sortablejs');
```
