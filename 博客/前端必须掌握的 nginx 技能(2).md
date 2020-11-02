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

[前端工程师学习 Nginx 入门篇](https://juejin.im/entry/56f23b77a34131005438d2e5)

### 设置反向代理

为什么叫反向代理？因为一般的代理是代理客户端，而如果我们要代理服务器的话，就好像反向一样，所以叫反向代理。所以我们把代理服务器的代理都叫反向代理。

nginx 里面设置反向代理只需要使用 proxy_pass 指令就可以了。具体文档在这里 [Module ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html)。示例如下：

```
events {
    # worker_connections  1024;  ## Default: 1024
}

http {
    server {
        listen  8767;
        server_name  192.168.2.32;
        location / {
            root /Users/zhouyang/Documents/tencent/test/local-nginx;
        }
        location /baidu {
            proxy_pass http://www.baidu.com;
        }
    }
}
```

我们在 bash 执行如下命令，然后打开 192.168.2.32:8767/baidu 就会自动跳转到百度。(注意：这里的 192.168.2.32 要改成你自己的 ip 地址)。

```
nginx -s quit // 优雅退出
nginx -c /Users/zhouyang/Documents/tencent/test/local-nginx/nginx.conf // 使用指令目录下的配置文件启动 nginx
```

### 设置跳转

1.根据 url 进行跳转。根据 url 进行跳转也很简单，代码改成如下即可：

```
events {
    # worker_connections  1024;  ## Default: 1024
}

http {
    server {
        listen  8767;
        server_name  192.168.2.32;
        location / {
            root /Users/zhouyang/Documents/tencent/test/local-nginx;
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

我们在 bash 执行如下命令，然后打开 192.168.2.32:8767/yaya ，就会自动跳转到 192.168.2.32:8767/yaya ，最后自动跳转到百度。(注意：这里的 192.168.2.32 要改成你自己的 ip 地址)。

```
nginx -s quit // 优雅退出
nginx -c /Users/zhouyang/Documents/tencent/test/local-nginx/nginx.conf // 使用指令目录下的配置文件启动 nginx
```

2.根据 referer 进行跳转。根据 url 进行跳转也很简单，代码改成如下即可。具体文档在这里 [Module ngx_http_referer_module](http://nginx.org/en/docs/http/ngx_http_referer_module.html)。

```
events {
    # worker_connections  1024;  ## Default: 1024
}

http {
    server {
        listen  8767;
        server_name  192.168.2.32;
        location / {
            root /Users/zhouyang/Documents/tencent/test/local-nginx;
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

我们在 bash 执行如下命令，然后打开 192.168.2.32:8767/haha ，就会看到 401 禁止访问了。(注意：这里的 192.168.2.32 要改成你自己的 ip 地址)。

注意：nginx 会根据 valid_referers 列出的进行匹配，如果匹配上则会自动赋值内部变量 $invalid_referer 为空字符串，如果没有匹配上则会自动赋值为字符串 '1'。

```
nginx -s quit // 优雅退出
nginx -c /Users/zhouyang/Documents/tencent/test/local-nginx/nginx.conf // 使用指令目录下的配置文件启动 nginx
```

### 根据 ip 地址控制访问

我们还可以根据 ip 地址控制访问，禁止某一些 ip 访问。示例如下：

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

上面我们禁止了 192.168.2.32 访问 192.168.2.32:8767，但是由于 nginx 的匹配机制，访问其它路由还是可以的。

### 负载均衡

我们还可以做负载均衡，具体文档在这里 [Module ngx_http_upstream_module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)。

我没有深入研究，想深入研究的话可以看文档和查资料。

### 添加 https

由于我还在申请 [aws ec2](https://amazonaws-china.com/cn/ec2/features/)，所以等申请完毕之后再回来补充。
