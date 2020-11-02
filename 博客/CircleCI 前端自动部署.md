## 概述

现在很多前端库都用到了 CircleCI 进行自动部署，比如[Vue](https://github.com/vuejs/vue)，[React](https://github.com/facebook/react)，作为一个前端我觉得还是有必要实操一下 CircleCI 的，总体来说还是挺简单的，我把过程和体会记录下来，供以后开发时参考，相信对其他人也有用。

### 步骤

1.首先登陆 [circleci](https://circleci.com/)，直接用 github 账号登录即可。登陆后点击右上角的 go to app。
2.进入网页版 app 之后，我们能看到一个 dashboard，然后点击左边导航栏的 add project。
3.然后选择需要自动部署的项目，比如我就是 [play-jest](https://github.com/sishenhei7/play-jest) 这个项目，点击 Set Up Project。
4.接着会让我们选 Operating System 和 Language，我们肯定选择 Linux 和 Node。
5.选好之后下面会告诉你接下来要怎么做，比如在项目里面创建一个```.circleci```文件夹，并在文件夹里面创建一个```config.yml```文件
6.然后可以复制官方给的```config.yml```文件模板，这里我稍微做一下改动，就是把 docker 的 circleci/node:7.10 改成 circleci/node:latest，使用最新的 node 版本。并上传到github。
7.最后点击 Start Building 就大功告成了。

### 为什么需要自动部署

除非满足下面二个条件之一，否则使用circleci自动部署完全没用:

1. 需要在每次提交代码之后自动运行测试文件。
2. 需要在每次提交代码之后自动生成部署。（比如自动生成生产环境代码，自动部署nginx等）

我记得之前使用 Hexo 博客的时候，每次提交 markdown 文件都需要重新构建一遍，非常麻烦，所以最后才转去用 React 写博客了。现在有了 CircleCI 自动部署，就没这么麻烦了。

或者在 github 上面搭建静态博客，每次提交代码之后就可以用 CircleCI 自动部署到 gh-page 分支，真的很方便。

### codecov

使用 CI 工具的时候顺便使用 codecov 上传覆盖率报告是最好不过的了。方法也很简单，只需要安装 codecov 这个库，然后在运行的代码后面加上 codecov 即可。比如说我，就在 package.json 里面使用如下的命令：

```
"test:coverage": "vue-cli-service test:unit --coverage --verbose && codecov"
```


