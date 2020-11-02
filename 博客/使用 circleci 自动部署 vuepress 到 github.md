## 概述

今天我想把博客什么的搬到 github 的 vuepress 上面。但是每次提交 md 文件需要手动打包然后再提交到 github 的 gh-pages，非常麻烦。所以我去研究了一下用 circleci 自动集成。总体来说还是比较简单的。我把新的记录下来，供以后开发时参考，相信对其他人也有用。

[我的 vuepress 博客地址(目前还没什么内容)](https://sishenhei7.github.io/vue-blog/)

### 集成步骤

1.把项目提交到 master 分支，然后在项目主目录下面创建 .circleci 文件夹，在文件夹里面创建 config.yml 文件，写入下列内容：

``` bash
# Javascript Node CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-javascript/ for more details
#
version: 2
jobs:
  build:
    docker:
      # specify the version you desire here
      - image: circleci/node:latest

      # Specify service dependencies here if necessary
      # CircleCI maintains a library of pre-built images
      # documented at https://circleci.com/docs/2.0/circleci-images/
      # - image: circleci/mongo:3.4.4

    # working_directory: ~/repo

    # only master branch will be deployed
    filters:
      branches:
        only: master

    steps:
      # connect to github by ssh
      - add_ssh_keys:
          fingerprints:
          - "xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:"

      - checkout

      # Download and cache dependencies
      - restore_cache:
          keys:
          - v1-dependencies-{{ checksum "package.json" }}
          # fallback to using the latest cache if no exact match is found
          - v1-dependencies-

      - run: yarn install

      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}

      - run: sh deploy-circleci.sh
```

简单来说，上面文件的意思是，每次 master 分支有修改的时候，就自动触发 circleci 集成功能，让它用 ssh 连接 github 部署到 gh-pages。

2.我们给 circleci 创建 ssh。进入这个地址 ```https://circleci.com/gh/<github_name>/<repo_name>/edit#checkout```。点击使用 user key，然后授权 circleci 自动生成 user key，最后点击 add user key就可以了。(这里设置了之后需要把 fingerprints 加到步骤一的文档里面去)

这里说明一下 deploy key 和 user key 的区别：

1. checkout 页面的 deploy key 只有读权限，没有写权限；如果要加入有读写权限的 deploy key，需要手动生成 ssh 然后到 ssh permission 页面去加。deploy key 只对**特定仓库**有权限。

2. user key 对你的账号下的**所有仓库**都有读写权限。

这里是[integration 文档](https://circleci.com/docs/2.0/gh-bb-integration/)，[github ssh 文档](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)


3.我们创建 circleci 的 deploy 文档。在主目录下面创建 deploy-circleci.sh 文件，内容如下：

``` bash
git pull
yarn build
git checkout gh-pages
git push
```

4.依据[CircleCI 前端自动部署](https://www.cnblogs.com/yangzhou33/p/11440979.html)的步骤集成到 circleci 上面去即可。
