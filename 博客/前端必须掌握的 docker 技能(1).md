## 概述

作为一个前端，我觉得必须要学会使用 docker 干下面几件事：

1. 部署前端应用
2. 部署 nginx
3. 给部署的 nginx 加上 https
4. 使用 docker compose 进行部署
5. 给 nginx 加上 redis
6. 使用 kubernetes

下面我按照这个节奏一一研究一遍，把心得记录下来，供以后开发时参考，相信对其他人也有用。

### 部署前端应用

(安装 docker 的过程就略去了，请自行查看官方文档安装)

1.我们设置国内镜像，方便以后下载镜像文件。

```
1.点击屏幕上方的鲸鱼 docker 图标。
2.点击 Preferences。
3.点击 Deamon。
4.在 Registry mirrors 那里填上 https://registry.docker-cn.com
```

2.拉取 node 镜像。输入下面的命令远程拉取最新版本的 node 镜像。

```
docker pull node:latest
```

查看本地镜像有哪些：

```
docker images
```

3.在前端项目的主目录下建立 Dockerfile 文件，写入如下内容：

```
FROM node:latest
WORKDIR /home/app
COPY . .
RUN npm install
expose 8080
CMD npm run serve

// 使用最新的 node 镜像
// 设置工作目录为 /home/app
// 把当前目录下的文件全部复制到工作目录下面
// 在工作目录下面运行 npm install
// 暴露 8080 端口(注意：这个 8080 端口就是 npm run serve 的端口)
// 在 cmd 里面使用 npm run serve 命令(注意：这里需要用你自己项目中的命令)
```

4.我们并不需要把 node_modules 里面的文件复制到 docker 里面去，所以我们在前端项目的主目录下建立 .dockerignore 文件，并写入如下内容：

```
node_modules
```

5.上面只是准备工作，现在才正式开始。我们使用刚才建立好的 Dockerfile 文件给项目进行打包，建立一个镜像文件：

```
docker build -t docker-app:latest .
```

docker 内部会生成一份这个镜像文件，我们可以用```docker images```查看。

6.用这个镜像文件创建一个实例并运行。

```
docker run -d -p 2001:8080 docker-app

// -d 意思是在后台运行
// -p 2001:8080 意思是把 docker 的 8080 端口连接到本机的 2001 端口
// 以后就可以通本机的 2001 端口访问 docker 的 8080 端口了
```

7.最后打开 localhost:2001 即可。

### 搞事

有时我们想**进入 docker 的 bash 里面**搞点事，那方法是运行如下命令即可：

```
docker exec -it [container_id] /bin/bash
```

那我们怎么得到这个容器 container_id 呢？ 运行如下命令即可：

```
docker ps -a
```

那我们要怎么删除一个镜像/容器呢？

```
docker rmi -f [image_id]
docker rm -f [container_id]
```

那我们要怎么停止/开始一个容器呢？

```
docker stop [container_id]
docker start [container_id]
```

注意：本文的步骤**显然不是正常部署的步骤**，因为我们在 docker 里面使用的是 npm run dev 命令啊。所以实际上部署只需要把 dist 文件夹里面的内容 copy 进 docker 镜像里面去就可以了。






