## 概述

一般来说，在终端开启的服务，如果退出终端的话，就会自动关闭服务。这个时候需要守护这个服务的进程。

Supervisor 是一个用 Python 写的进程管理工具，可以很方便的用在 UNIX-like 系统（不支持 Windows ）下启动、重启、关闭进程等。其中 **supervisord** 是 server 端，**supervisorctl** 是 client 端。

参考资料：

[Supervisor管理项目、进程](https://blog.csdn.net/livetxl/article/details/79012784)
[python web 部署：nginx + gunicorn + supervisor + flask 部署笔记](https://www.jianshu.com/p/be9dd421fb8d)

### 安装

直接用 pip 安装即可

``` bash
sudo pip install  supervisor
```

安装之后就可以在终端中执行这三个命令：**echo_supervisord_conf(生成配置文件)，supervisord(服务端)，supervisorctl(客户端)**。

### echo_supervisord_conf

我们用 echo_supervisord_conf 命令生成**配置文件**：

``` bash
echo_supervisord_conf > supervisor.conf   # 生成 supervisor 默认配置文件
```

然后我们可以在配置文件中部署一个 flask 应用，示例如下：

```
[program:myapp]
command=/Users/zhouyang/Documents/projects/flask-project/venv/bin/gunicorn -w4 -b0.0.0.0:2171 hello:app    ; supervisor启动命令
directory=/Users/zhouyang/Documents/projects/flask-project              ; 项目的文件夹路径
startsecs=0              ; 启动时间
stopwaitsecs=0           ; 终止等待时间
autostart=false          ; 是否自动启动
autorestart=false        ; 是否自动重启
stdout_logfile=/Users/zhouyang/Documents/projects/flask-project/log/gunicorn.log     ; log 日志
stderr_logfile=/Users/zhouyang/Documents/projects/flask-project/log/gunicorn.err
```

需要注意的是，我是用的 venv **虚拟环境**的 flask。

### supervisord

然后我们通过配置文件启动 Supervisor 服务端：

``` bash
supervisord -c supervisor.conf    # 通过配置文件启动supervisor
```

### supervisorctl

启动服务端之后，我们查看进程运行状态：

``` bash
supervisorctl -c supervisor.conf status        # 察看supervisor的状态
```

我们发现进程并没有运行，因为需要我们手动让进程运行：

``` bash
supervisorctl -c supervisor.conf start [all]|[appname]     # 启动指定/所有 supervisor管理的程序进程
supervisorctl -c supervisor.conf stop [all]|[appname]      # 关闭指定/所有 supervisor管理的程序进程
```

如果要添加新的进程，需要改配置文件，然后重启服务器：

``` bash
supervisorctl -c supervisor.conf reload        # 重新载入 配置文件
```

### 管理界面

最后，supervisor 还有一个 web 的**管理界面**，需要我们在配置文件里面修改成下面这样：

```
[inet_http_server]         ; inet (TCP) server disabled by default
port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
username=user              ; (default is no username (open server))
password=123               ; (default is no password (open server))

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
username=user              ; should be same as http_username if set
password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available
```

然后我们访问 http://127.0.0.1:9001 可以看到 supervisor 的 web 管理界面啦~~
