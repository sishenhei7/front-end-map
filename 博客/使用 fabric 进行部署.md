## 概述

前端打包完成之后需要上传到服务器，怎么上传呢？可以先上传到 github，然后在远程服务器上面拉取，最后打包上线。但是这样很麻烦，使用 fabric 可以很简单的一键部署。我根据自己的使用经验，把 fabric 的部署过程记录下来，供以后开发时参考，相信对其他人也有用。

注意：我使用的是 **fabric3**

参考文件：

[fabric 文档](https://fabric-chs.readthedocs.io/zh_CN/chs/tutorial.html)
[fabric operations](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/operations.html)

### fabric 介绍

fabric3 的安装略，请自己 google。

fabric3 我们主要使用这三个模块：[业务（Operation）](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/operations.html) 、[上下文管理器](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/context_managers.html) 、 [装饰器](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/decorators.html) 以及 [实用工具](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/utils.html)。

### 业务（Operation）模块

业务模块中有我们需要在 fabric 里面运行的各种函数，比如 run()/sudo() 等。

我们使用下面的方式引入业务模块中的函数(local, put, run等)：

```python
from fabric.operations import local, put, run
```

下面我们来简要介绍一下业务模块中的函数，如果想要了解具体使用方法，请查看[文档](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/operations.html)

1. get(): 从远程主机下载一个或多个文件
2. local(): 在本地 bash 里面运行一个命令
3. run(): 在远程主机上执行 shell 命令
4. sudo(): 在远程主机上使用超级用户权限执行 shell 命令
5. put(): 上传文件到远程主机上面
6. open_shell(): Invoke a fully interactive shell on the remote end.
7. prompt(): Prompt user with text and return the input
8. reboot(): Reboot the remote system.
9. require(): Check for given keys in the shared environment dict and abort if not found.

注意，如果 run() 加了 quiet=True 则表示隐藏全部输出，示例如下：

```bash
run('mkdir deploy', quiet=True)
```

### 上下文管理器

上下文管理器模块中有我们切换上下文所需要的各种函数，比如 cd()/prefix() 等。

我们使用下面的方式引入上下文管理器模块中的函数(cd等)：

```python
from fabric.context_managers import cd
```

下面我们来简要介绍一下上下文管理器模块中常用的函数，如果想要了解具体使用方法，请查看[文档](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/context_managers.html)

1. cd(): 切换远程主机上的路径，所有在 cd() 函数里面的 run, sudo, get, or put 都会被自动加上 ```"cd <path> && "``` 前缀。
2. lcd(): 类似 cd()，只不过 cd() 适用于远程主机；lcd() 适用于本地。
3. prefix(): Prefix all wrapped run/sudo commands with given command plus &&

示例如下：

```bash
with cd('/path/to/app'):
    with prefix('workon myvenv'):
        run('./manage.py syncdb')
        run('./manage.py loaddata myfixture')
```

相当于：

```bash
$ cd /path/to/app && workon myvenv && ./manage.py syncdb
$ cd /path/to/app && workon myvenv && ./manage.py loaddata myfixture
```

### 装饰器模块

装饰器模块中有我们可以使用的各种装饰器，比如 hosts() 等。

我们使用下面的方式引入装饰器模块中的函数(hosts等)：

```python
from fabric.api import hosts
```

下面我们来简要介绍一下装饰器模块中的常用函数，如果想要了解具体使用方法，请查看[文档](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/decorators.html)

1. hosts(): 该装饰器用于指定被装饰的函数执行在那台主机或哪些主机列表上。

注意：如果不在控制台覆盖相关参数的话，将会在 host1、host2 以及 host3 上执行 my_func，并且在 host1 和 host3 上都指定了登录用户。示例如下：

```
@hosts('user1@host1', 'host2', 'user2@host3')
def my_func():
    pass
```


### 实用工具

实用工具模块中有我们可以使用的各种工具函数，比如 abort() 等。

我们使用下面的方式引入实用工具模块中的函数(abort)：

```python
from fabric.api import abort
```

下面我们来简要介绍一下实用工具模块中的常用函数，如果想要了解具体使用方法，请查看[文档](https://fabric-chs.readthedocs.io/zh_CN/chs/api/core/utils.html)

1. abort(): 终止执行，向 stderr 输入错误信息 msg 并退出

示例如下：

```
from __future__ import with_statement
from fabric.api import local, settings, abort
from fabric.contrib.console import confirm

def test():
    with settings(warn_only=True):
        result = local('./manage.py test my_app', capture=True)
    if result.failed and not confirm("Tests failed. Continue anyway?"):
        abort("Aborting at user request.")
```

### fabric 文件示例

下面是 fabric 文件的示例：

```
# -*- coding: utf-8 -*-

from fabric.api import hosts
from fabric.context_managers import cd
from fabric.operations import run


@hosts('xxxxxxxxxxxx')
def deploy():
    project_dir = 'xxxxxxxxxxxx'
    deploy_project_dir = 'xxxxxxxxxxxx'
    with cd(project_dir):
        run('git pull')
        run('npm run build')
        with cd(deploy_project_dir):
            run(f'cp -r {project_dir}/dist dist_staging')
            run('rm -rf dist_bak')
            run('mv dist dist_bak')
            run('rm -rf dist')
            run('mv dist_staging dist')

def fallback():
    deploy_project_dir = 'xxxxxxxxxxxxxxxx'
    with cd(deploy_project_dir):
        run(f'cp -r dist_bak dist_staging')
        run('rm -rf dist_bak')
        run('mv dist dist_bak')
        run('rm -rf dist')
        run('mv dist_staging dist')
```

需要注意的是:

1. 上面的代码对打包后的文件做了备份，然后可以通过使用备份来回退
2. 上面的代码在远程主机进行打包的，也可以在本地打包然后用 put() 上传
3. 可以在 run() 里面开启静默模式


然后直接在当前目录里面运行下列命令就可以使用啦：

```
fab deploy
fab fallback
```

### fabric 的一个坑

fabric 读取环境变量的时候**只会读 .bash_profile 里面的环境变量**，所以如果在执行 fabric 的时候碰到 没有xxx命令 的情况，就把相关的环境变量写到 .bash_profile 里面去吧~~~
