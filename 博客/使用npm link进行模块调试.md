## 概述

我还记得之前调试官网的 ckeditor 编辑器，每次改完编辑器包，然后发布编辑器包，然后在官网里面引入这个包进行调试，如果有问题，重新改动、发布编辑器包，继续引入，真的浪费了好多时间。其实类似这种场景都可以使用[npm link](https://docs.npmjs.com/cli/link)，能够极大地简化操作。记录下来，供以后开发时参考，相信对其他人也有用。

### 直接使用 npm link

比如说有包 package-A 和项目 project-B，项目 project-B 需要用到包 package-A。而他们都是在 projects 文件夹里面，那么我们可以直接在 project-B 里面 npm link，示例如下：

```
cd ~/projects/project-B  # go into the dir of project B
npm link ../package-A    # link the dir of your package
```

注意：link完之后，就相当于把这个包放到项目的 node_modules 里面去了，你可以直接在项目中引用这个包。

### 先把 package link 为全局，再在项目中引入

还是包 package-A 和项目 project-B，如果他们放在不同的文件夹里面，并且相对路径不太方便写，或者他们的位置会变动，那么可以使用如下方式：

```
cd ~/packages/package-A
npm link                   # creates global link
cd ~/projects/project-B
npm link package-A         # link-install the package
```

注意：link完之后，就相当于把这个包放到项目的 node_modules 里面去了，你可以直接在项目中引用这个包。

### 其它

1.如果是全局 link 的方式，也会把 bins 文件夹放到全局，所以也可以让某些命令全局化。

2.npm link 的原理是建立 symlink(软链接)，但是并不是以文件夹名字的形式，而是以```package.json```里面包的名字的形式。
