## 概述

这几天玩 jest ，我在运行单元测试之后 coverage 总是显示 unknown，花了很多时间排查原因，最后终于想明白了，记录下来，供以后开发时参考，相信对其他人也有用。

### coverage参数

首先最可能的原因是，命令中没有带```--coverage```参数。一般 github 的 issue 里面都是说的这个原因。

但是我玩的不是原生jest，而是 vue-cli 的 @vue/cli-plugin-unit-jest 包里面 jest ，包里面已经帮我们配置好了一些参数，包括这个```--coverage```参数。我们的package.json文件里面是这么写的：

```
"scripts": {
  "test:unit": "vue-cli-service test:unit --coverage"
},
```

所以当执行```npm run test:unit```的时候，会自动带上```--coverage```参数。

### coverage 需要测试文件

另一个原因是，我们没有指明需要coverage的文件名。

测试，顾名思义，就是测试文件，如果只有spec.js测试文件，而没有被测试的文件，当然就没有coverage了。所以加上被测试的文件即可。

比如报错的时候我的目录结构是：

```
-tests
  -unit
    -testMatchers.spec.js
```

后来我把被测试文件放到一个专门的文件夹里面就可以，修复后的文件目录是：

```
-tests
  -helper
    -testMatchers.js
  -unit
    -testMatchers.spec.js
```

所以被测试的文件就是```./helper/testMatchers.js```，而coverage的覆盖率指的是对这个文件测试的覆盖率，并不是```./unit/testMatchers.js```这个文件的覆盖率哦~~


