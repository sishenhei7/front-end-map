## 概述

目前，[Let's Encrypt](https://letsencrypt.org/) 可以算是最好用的 https 证书申请网站了吧。而 [certbot](https://certbot.eff.org/) 可以算是它的客户端，能够很方便的自动生成 https 证书。我把自己的使用经历记录下来，供以后开发时参考，相信对其他人也有用。

### 操作步骤

1.首先我们去 [certbot](https://certbot.eff.org/) 官网选择自己的服务器和操作系统。我使用的是CentOS/RHEL7 上的 nginx。

2.然后按照官方给出的安装步骤一步步安装。这是教程地址：[centosrhel7-nginx](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx)

### 几个坑

1.怎么**测试服务器的某个端口是否打开**。方法是使用 telnet 或者 nc。代码如下：

```bash
## 使用 telnet + ip + 端口号
telnet xxx.xxx.xxx.xxx xx

## 使用 nc
nc  -z xxx.xxx.xxx.xxx xx    ## tcp 端口
nc  -uz xxx.xxx.xxx.xxx xx   ## udp 端口
```

2.在配置 certbot 的时候意外退出ssh，再进去配置的时候提示```Another instance of Certbot is already running```的解决方法：

```
find / -type f -name ".certbot.lock" -exec rm {} \;
```

3.服务器可能有安全组设置，所以如果按照上面的顺序配置还是无效的话 ，检查下服务器的**防火墙和安全组设置**。
