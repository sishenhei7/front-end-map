## 概述

作为一个前端，我觉得必须要学会使用 docker 干下面几件事：

1. 部署前端应用
2. 部署 nginx
3. 给部署的 nginx 加上 https
4. 使用 docker compose 进行部署
5. 给 nginx 加上 redis
6. 使用 kubernetes

下面我按照这个节奏一一研究一遍，把心得记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[docker nginx](https://docs.docker.com/samples/library/nginx/)
[Beginner’s Guide](http://nginx.org/en/docs/beginners_guide.html#control)

**注意：因为我正在申请域名和云服务器，所以暂时还不能给部署的 nginx 加上 https。**

### 使用 docker compose 进行部署

在前面我们使用 docker 部署了一个 nodejs 的应用，还部署了一个 nginx。启动的时候还需要分开启动，非常麻烦，其实我们可以一键进行部署和启动的。这就是 docker-compose。

首先来说下我们的目标：

1. 使用 docker-compose **同时部署** nodejs 和 nginx
2. 挂载 src 目录，实现**本地开发 src，远程自动热更新**（本地不需要搭建环境）
3. 挂载 nginx 配置，实现**本地修改 nginx 配置**

### 步骤

1.作为示例，我们使用使用以下命令创建一个新的 vue 项目：

```
vue create vue-demo
```

2.在主目录文件夹下面建立 Dockerfile 和 .dockerignore 文件：

``` bash
// Dockerfile
FROM node:latest
WORKDIR /app/vue-demo
COPY . /app/vue-demo
RUN npm install
CMD npm run serve

// .dockerignore
node_modules
```

3.编写 docker-compose 文件，示例如下：

``` bash
version: '3.7'
services:
  web:
    build:
      context: ./
      dockerfile: ./Dockerfile
    image: "vue-demo"
    ports:
      - "5431:8080"
    volumes:
      - ./Volumes/src:/app/vue-demo/src
  nginx:
    image: "nginx"
    ports:
      - "5432:80"
    volumes:
      - ./Volumes/default.conf:/etc/nginx/conf.d/default.conf

// version 表示用什么版本的 docker-compose
// web 和 nginx 是我们自己取的名字
// 我们用 build 来指定 dockerfile，注意这里需要先指定 context 为主目录
// 我们指定本机的 5431 端口连接 docker 的 8080 端口(8080 是 vue-demo 启动 npm run serve 时的端口)
// 我们挂载 Volumes 文件夹下的 src 到 docker 的 src(/app/vue-demo 是在 Dockerfile 里面写的目录)
// nginx 我们使用 nginx 镜像，而不是用 Dockerfile 生成
// 我们指定本机的 5432 端口 连接 docker 的 80 端口(80 端口是 nginx 代理的端口)
// 我们挂载 Volumes 文件夹下的 default.conf 到 docker 的 default.conf
```

注意：这里需要**新建一个 Volumes 文件夹(挂载目录)**，然后在里面放入挂载文件。否则需要在 docker 设置里面去增加挂载目录。

进行上面的设置之后，我们：

1. 修改了 Volumes/src 文件夹之后，```0.0.0.0:5431```能够自动热重载进行更新。
2. 我们可以通过修改 Volumes/default.conf 来调整 nginx 的配置。

注意：我们这里部署的是一个**开发环境**，其实也可以部署一个**发布环境**，这个比较简单只需要挂载 dist 文件夹就行了。

4.把 src 文件夹复制到 Volumes 文件夹下面并且在里面新建 default.conf 文件。最后组装并运行。

``` bash
// 有 bash 命令行
docker-compose up

// 无 bash 命令行
docker-compose up -d
```

通过 docker ps 查看正在运行的 container，我们可以看到我们已经部署成功了。


