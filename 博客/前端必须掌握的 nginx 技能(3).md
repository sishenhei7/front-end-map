## 概述

作为一个前端，我觉得必须要学会使用 nginx 干下面几件事：

1. 代理静态资源
2. 设置反向代理（添加https）
3. 设置缓存
4. 设置 log
5. 部署 smtp 服务
6. 设置 redis 缓存（选）

下面我按照这个节奏一一研究一遍，把心得记录下来，供以后开发时参考，相信对其他人也有用。

### 设置缓存

缓存一般是设置在 location 块里面，示例代码如下：

```
events {
    # worker_connections  1024;  ## Default: 1024
}

http {
    server {
        listen  8767;
        server_name  192.168.2.32;
        location / {
            deny 192.168.2.32;
            root /Users/zhouyang/Documents/tencent/test/local-nginx;
            expires      30s;
        }
        location /haha {
            valid_referers none blocked server_names
               *.example.com example.* www.example.org/galleries/
               ~\.google\.;

            if ($invalid_referer = '') {
                return 401;
            }
        }
        location /baidu {
            proxy_pass http://www.baidu.com;
        }
        location /yaya {
            return 302 /baidu;
        }
    }
}
```

其中 ```expires 30s;``` 就是设置缓存为 30 秒。expire 指令的单位如下：

```
expires 30s; #30秒
expires 30m; #30分钟
expires 2h; #2个小时
expires 30d; #30天
```

如果不需要设置缓存，则改成如下代码：

```
expires -1s;
add_header Cache-Control no-cache;
```

通过浏览器查看请求的详细信息，我们可以看到：

```
// 设置缓存多了如下字段
Cache-Control: max-age=30
Expires: Fri, 27 Sep 2019 01:00:47 GMT

// 取消缓存多了如下字段
Cache-Control: no-cache;
Expires: Fri, 27 Sep 2019 00:57:40 GMT;
```

注意：在 vue 项目中，我们不建议对 html 设置缓存，但是我们建议对 js，css 文件设置缓存，因为我们打包的时候就已经加了 hash 了，所以即使文件变动，也会是新的文件名，不是老的文件名。我们可以利用 location 里面的 if 控制实现。

### 设置 Gzip 压缩

想要开启 Gzip 压缩，只需要加上如下代码即可：

```
gzip on;                     # 开启Gzip
gzip_min_length  1k;         # 不压缩临界值，大于1K的才压缩
gzip_types text/plain text/css application/x-javascript application/javascript application/xml;   # 哪些类型需要压缩
```

经过测试，上面这段 Gzip 的代码可以**加在 http 指令块、server 指令块甚至 location 指令块里面**。可以按各自的需求进行配置。




