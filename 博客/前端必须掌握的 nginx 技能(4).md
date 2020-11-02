## 概述

作为一个前端，我觉得必须要学会使用 nginx 干下面几件事：

1. 代理静态资源
2. 设置反向代理（添加https）
3. 设置缓存
4. 设置 log
5. 部署 smtp 服务
6. 设置 redis 缓存（选）

下面我按照这个节奏一一研究一遍，把心得记录下来，供以后开发时参考，相信对其他人也有用。

### 设置 log

nginx 的日志分为访问日志和错误日志，在配置中加入如下命令即可开启日志：

```
access_log /Users/zhouyang/Documents/tencent/test/local-nginx/nginx-access.log;
error_log /Users/zhouyang/Documents/tencent/test/local-nginx/error.log;
```

其中后面的是日志存放的路径。

### 生产环境日志配置

我们一般使用变量来定制日志输出的格式，示例如下：

```
events {
    # worker_connections  1024;  ## Default: 1024
}

http {
    server {
        listen  8767;
        server_name  192.168.2.32;
        location / {
            gzip  on;
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
    log_format  main    '$host $remote_addr - [$time_local] "$request" $status '
                        '$body_bytes_sent "$http_referer" "$http_user_agent" '
                        '[$request_time $upstream_response_time]';

                        ## '' 是对多个变量的囊括
                        ## - [] 等会被当做分隔符打印在变量中间
                        ## $host 域名
                        ## $remote_addr 客户端地址
                        ## $time_local 时间
                        ## $request 请求头信息
                        ## $status 返回状态
                        ## $body_bytes_sent 响应的 body 的大小
                        ## $http_referer 上一级访问的地址,不是客户端上一次访问的页面地址
                        ## $http_user_agent 访问的客户端类型（浏览器，curl，等）
                        ## $request_time 请求处理时间
                        ## $upstream_response_time 后端处理时间

    access_log /Users/zhouyang/Documents/tencent/test/local-nginx/nginx-access.log main;
    error_log /Users/zhouyang/Documents/tencent/test/local-nginx/error.log;
}
```
