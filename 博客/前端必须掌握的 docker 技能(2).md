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

### 部署 nginx

1.拉取 nginx 镜像。输入下面的命令远程拉取最新版本的 nginx 镜像。

```
docker pull nginx:latest
```

2.在前端项目的主目录下建立 Dockerfile 文件，写入如下内容：

```
FROM nginx
```

3.生成镜像

```
docker build -t docker-nginx:latest .
```

4.运行镜像实例

```
docker run -d -p 2002:80 docker-nginx
```

5.最后打开 localhost:2001 即可看到 nginx 标准的欢迎界面。

### 搞事

如果我们想**自定义 docker 里面的 nginx 的配置文件**呢？

1.我们进入 docker-nginx 容器的 bash 界面

```
docker exec -it [container_id] /bin/bash
```

2.查看 nginx 的配置文件夹路径

```
nginx -t

// 输出如下内容
// nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
// nginx: configuration file /etc/nginx/nginx.conf test is successful
```

可以看到配置文件放在 /etc/nginx/nginx.conf。

3.查看配置文件内容

```
cat /etc/nginx/nginx.conf

// 在最下面可以看到这么一段
// include /etc/nginx/conf.d/*.conf;
```

在最下面可以看到这么一段 ```include /etc/nginx/conf.d/*.conf;```，也就是说副配置文件放在了 /etc/nginx/conf.d/default.conf。我们打算改写这个副配置文件。

4.改写 Dockerfile 文件如下所示：

```
FROM nginx
COPY default.conf /etc/nginx/conf.d/default.conf
```

即是说，用当前目录下的 default.conf 文件替换 docker 里面的 default.conf。

5.最后重新生成一遍镜像并运行容器即可。

注意：我这里的方法**并不是最优的**，还有挂载配置文件，使用 docker compose 方法比这个好得多，以后再介绍。





