## 概述

今天我发现我的所有项目的 CircleCi 部署**全部都会更新 gh-pages 分支**。找了好久，终于找到了不更新的方法。于是我总结了一下，记录下来，供以后开发时参考，相信对其他人也有用。

### only 方法

其实在 config.yml 文件的 build 里面**加入 branches 字段**，就会只更新 master 分支，示例如下：

```
version: 2
jobs:
  build:
    branches:
      only:
          - master

    docker:
      # specify the version you desire here
      - image: circleci/node:latest
      # 其它配置
      # ...
```

如果不想更新 xxx 分支的话，可以使用下面的写法：

```
version: 2
jobs:
  build:
    branches:
      ignore:
          - xxx
          - xxx

    docker:
      # specify the version you desire here
      - image: circleci/node:latest
      # 其它配置
      # ...
```

### ci skip

但是上面的写法**不适用于 gh-pages 分支**。如果要 CircleCi 自动更新 gh-pages 分支，但是这个更新不触发 CircleCi 的话，需要在部署脚本的 commit 信息里面加入 ```[ci skip]```，示例如下：

```
git pull
yarn build:doc
cd docs
git init
git config --global user.email "you@example.com"
git config --global user.name "sishenhei7"
git add .
git commit -m 'deploy: build docs[ci skip]'
git push -f git@github.com:sishenhei7/digit-flip.git  master:gh-pages
cd -
```

