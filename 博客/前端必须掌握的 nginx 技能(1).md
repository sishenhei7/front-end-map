## 概述

作为一个前端，我觉得必须要学会使用 nginx 干下面几件事：

1. 代理静态资源
2. 设置反向代理（添加https）
3. 设置缓存
4. 设置 log
5. 部署 smtp 服务
6. 设置 redis 缓存（选）

下面我按照这个节奏一一研究一遍，把心得记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[nginx 基本入门](https://segmentfault.com/a/1190000007683133)

[Beginner’s Guide](http://nginx.org/en/docs/beginners_guide.html)

### nginx 重要点

（nginx 的安装我就不介绍了，自己按文档安装就行）

1.如果 nginx 已经开启，那么可以使用如下命令控制 nginx

```
nginx -s signal

// 其中 signal 是如下命令：
// stop — 直接关闭 nginx
// quit — 会在处理完当前正在的请求后退出，也叫优雅关闭
// reload — 重新加载配置文件，相当于重启
// reopen — 重新打开日志文件
```

2.nginx 配置文件的语法是有简单指令和块级指令构成的：

```
// 简单指令由名字和参数组成，中间用空格分开，并以分好结尾，示例如下
root /data/www;

// 块级指令也叫上下文，用 { 和 } 大括号包裹，末尾没有分号，示例如下
// 其中注释以 # 开头
events {
  worker_connections  4096;  ## Default: 1024
}
```

注意：没有放在任何上下文中的指令都是处在主上下文中。events 和 http 的指令是放在主上下文中，server 放在 http 中, location 放在 server 中。结构示例如下：

```
events {

}

http {
  server {
    location / {

    }
  }
}
```

3.检测配置文件，查看配置文件的位置：

```
nginx -t

// 返回如下：
// nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
// nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```

### 代理静态资源

我们现在准备使用本机的 nginx 代理静态资源。

1.随便建立一个文件夹，在里面创建 index.html 和 nginx.conf。我们准备使用 nginx.conf 修改配置，然后代理 index.html。

2.在 index.html 里面写入如下代码：

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    hello world
</body>
</html>
```

3.在 nginx.conf 里面写入如下代码：

``` bash
events {
    # worker_connections  1024;  ## Default: 1024
}

http {
    server {
        listen  8765;
        location / {
            root /Users/zhouyang/Documents/tencent/test/local-nginx;
        }
    }
}
```

需要注意如下3点：

1. root 那里不能使用相对路径，因为我们是改写 /usr/local/etc/nginx/nginx.conf，所以相对路径的相对位置并不是当前所在的文件夹，而是 /usr/local/etc/nginx/ 文件夹。获取当前文件夹绝对路径的方法是：直接把此文件夹拖到 bash 里面即可。

2. 如果报错：```nginx: [emerg] "server" directive is not allowed here in xxxxxx```，意思是说 server 位置有误，它需要被放在 http 上下文里面！！

3. 如果报错：```nginx: [emerg] no "events" section in configuration```，意思是说没有 events 上下文，这里配置文件中必须加上 events 上下文，即使里面什么指令也没有。（就像上面我把 events 里面的内容注释掉了一样）

4.在 bash 里面使用如下命令修改 nginx 配置，然后重启 nginx。

```
// 首先优雅退出 nginx
nginx -s quit

// 然后从选定的配置文件启动 nginx
nginx -c /Users/zhouyang/Documents/tencent/test/local-nginx/nginx.conf
```

注意：第二步不能加 -t 参数写成```nginx -t -c /Users/zhouyang/Documents/tencent/test/local-nginx/nginx.conf```，因为 -t 参数只是检查配置，并且不启动 nginx。

5.打开 localhost:8765，即可看到 hello world。
